function protectFormulaColumns() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const classSheets = ["class1","class2","class3","class4","class5","class6","class7","class8","class9","class10","class11","class12"];
  const columnsToProtect = ["I","L","M","T","AJ","AK","AL","AZ","BA","BB","BP","BQ","BR","BS","BT","BU"];
  
  classSheets.forEach(sheetName => {
    const sheet = ss.getSheetByName(sheetName);
    if (!sheet) return;
    
    columnsToProtect.forEach(col => {
      const range = sheet.getRange(col + ":" + col);
      const protection = range.protect();
      protection.setDescription(`Protected formulas in ${col} (${sheetName})`);
      protection.removeEditors(protection.getEditors());
      // Allow only owner (you)
      if (protection.canDomainEdit()) protection.setDomainEdit(false);
    });
  });
  
  Logger.log("Formula columns protected successfully!");
}




function onEdit(e) {
  if (!e) return; // Safety for manual runs
  const sheet = e.range.getSheet();
  const sheetName = sheet.getName();
  const row = e.range.getRow();
  const col = e.range.getColumn();

  // Run only if editing class sheets
  const allowedSheets = [
    "class1","class2","class3","class4","class5","class6",
    "class7","class8","class9","class10","class11","class12"
  ];
  if (!allowedSheets.includes(sheetName)) return;
  if (row < 6) return;

  // === 1. H = H + G when G edited ===
  if (col === 7) {
    const valG = Number(sheet.getRange(row, 7).getValue()) || 0;
    const currentH = Number(sheet.getRange(row, 8).getValue()) || 0;
    sheet.getRange(row, 8).setValue(currentH + valG);
  }

  // === 2. Q = Q + P when P edited ===
  if (col === 16) {
    const valP = Number(sheet.getRange(row, 16).getValue()) || 0;
    const currentQ = Number(sheet.getRange(row, 17).getValue()) || 0;
    sheet.getRange(row, 17).setValue(currentQ + valP);
  }

  // === 3. Track G column and put date in J ===
  if (col === 7) {
    const valG = sheet.getRange(row, 7).getValue();
    const cellJ = sheet.getRange(row, 10);
    const existingJ = cellJ.getValue().toString().trim();
    const today = Utilities.formatDate(new Date(), Session.getScriptTimeZone(), "yyyy-MM-dd");

    if (valG === 0) {
      if (existingJ === "") cellJ.setValue(today);
      else {
        const parts = existingJ.split(",").map(s => s.trim());
        if (!parts.includes(today)) cellJ.setValue(existingJ + ", " + today);
      }
    }
  }
  // === Auto-calc I when H edited (I = H4 - H) ===
if (col === 8) {  // H column
  const H4 = Number(sheet.getRange(4, 8).getValue()) || 0;
  const valH = Number(sheet.getRange(row, 8).getValue()) || 0;
  sheet.getRange(row, 9).setValue(valH === "" ? "" : H4 - valH); // I column
}

// === Auto-calc L & M when K edited (L = L4*K , M = (L4*12) - L) ===
if (col === 11) {  // K column
  const L4 = Number(sheet.getRange(4, 12).getValue()) || 0; // L4
  const valK = Number(sheet.getRange(row, 11).getValue()) || 0;
  const calcL = (L4 * valK);
  sheet.getRange(row, 12).setValue(valK === "" ? "" : calcL); // L column
  sheet.getRange(row, 13).setValue(valK === "" ? "" : (L4 * 12) - calcL); // M column
}

// === Auto-calc T when S edited (T = S4 - S) ===
if (col === 19) { // S column
  const S4 = Number(sheet.getRange(4, 19).getValue()) || 0;
  const valS = Number(sheet.getRange(row, 19).getValue()) || 0;
  sheet.getRange(row, 20).setValue(valS === "" ? "" : S4 - valS); // T column
}


  // === 4. (K×L4 difference) appended to N when K edited ===
  if (col === 11) {
    const l4 = Number(sheet.getRange(4, 12).getValue()) || 0; // L4
    const oldK = Number(e.oldValue) || 0;
    const newK = Number(e.value) || 0;
    const diff = (newK * l4) - (oldK * l4);

    const today = Utilities.formatDate(new Date(), Session.getScriptTimeZone(), "yyyy-MM-dd");
    const entry = diff + "₹/" + today;

    const cellN = sheet.getRange(row, 14);
    const existingN = cellN.getValue().toString().trim();
    cellN.setValue(existingN ? existingN + ", " + entry : entry);

    const cellR = sheet.getRange(row, 18);
    const currR = Number(cellR.getValue()) || 0;
    cellR.setValue(diff);
  }

  // === 5. Update only this class summary in swader ===
  try { updateSingleClassSummary(sheetName); } 
  catch (err) { Logger.log("updateSingleClassSummary error: " + err); }
}


/**
 * Robust single-class summary updater.
 * Call with sheetName like "class7".
 */
function updateSingleClassSummary(sheetName) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var swader = ss.getSheetByName("swader");
  var sheet = ss.getSheetByName(sheetName);
  if (!swader || !sheet) return;

  var classNum = Number(sheetName.replace(/^class/i, ""));
  if (!classNum || classNum < 1 || classNum > 12) return;
  var swaderCol = 2 + (classNum - 1) * 2; // B=2, D=4, ...

  // Find a reliable last row by checking several important columns
  function lastRowOfColumn(col) {
    var vals = sheet.getRange(1, col, sheet.getMaxRows(), 1).getValues();
    for (var i = vals.length - 1; i >= 0; i--) {
      if (vals[i][0] !== "" && vals[i][0] !== null && typeof vals[i][0] !== "undefined") return i + 1;
    }
    return 0;
  }

  // Columns we care about (A=1, G=7, H=8, L=12, M=13, O=15, P=16, R=18, AL,BB,BR)
  var colLettersToNums = function(letter) { return sheet.getRange(letter + "1").getColumn(); };
  var colsToCheck = [1,7,8,12,13,15,16,18,
                      colLettersToNums("AL"),
                      colLettersToNums("BB"),
                      colLettersToNums("BR")];

  var lastRow = 0;
  for (var i = 0; i < colsToCheck.length; i++) {
    var lr = lastRowOfColumn(colsToCheck[i]);
    if (lr > lastRow) lastRow = lr;
  }

  // If no data at all (lastRow < 6) -> write zeros and return
  if (lastRow < 6) {
    swader.getRange(3, swaderCol).setValue(0);  // S no-zero
    swader.getRange(4, swaderCol).setValue(0);  // S with-zero
    swader.getRange(6, swaderCol).setValue(0);  // sum G
    swader.getRange(7, swaderCol).setValue(0);  // attendance %
    swader.getRange(8, swaderCol).setValue(0);  // sum L
    swader.getRange(9, swaderCol).setValue(0);  // sum M
    swader.getRange(10, swaderCol).setValue(0); // sum O
    swader.getRange(11, swaderCol).setValue(0); // sum P
    swader.getRange(12, swaderCol).setValue(0); // sum R
    swader.getRange(13, swaderCol).setValue(0); // AL pass %
    swader.getRange(14, swaderCol).setValue(0); // BB pass %
    swader.getRange(15, swaderCol).setValue(0); // BR pass %
    return;
  }

  var startRow = 6;
  var dataRowCount = lastRow - startRow + 1;

  // Safe getter that returns 2D array or empty array
  function getRangeValuesSafe(r, c, rows, cols) {
    if (rows <= 0) return [];
    return sheet.getRange(r, c, rows, cols).getValues();
  }

  // Sum helper for a single-column 2D array
  function sumNumeric(values2d) {
    var s = 0;
    for (var i = 0; i < values2d.length; i++) {
      var n = Number(values2d[i][0]);
      if (!isNaN(n)) s += n;
    }
    return s;
  }

  // ----- Count students in column A (S1, S01) -----
  var colAvals = getRangeValuesSafe(startRow, 1, dataRowCount, 1);
  var out_s_nozero = 0, out_s_withzero = 0;
  for (var r = 0; r < colAvals.length; r++) {
    var v = (colAvals[r][0] || "").toString().trim();
    if (!v) continue;
    if (/^S0+\d+$/i.test(v)) out_s_withzero++;
    else if (/^S[1-9]\d*$/i.test(v)) out_s_nozero++;
  }
  var totalStudents = out_s_nozero + out_s_withzero;

  // ----- Sums for G,H,L,M,O,P,R -----
  var out_sumG = sumNumeric(getRangeValuesSafe(startRow, 7, dataRowCount, 1));
  var sumH    = sumNumeric(getRangeValuesSafe(startRow, 8, dataRowCount, 1));
  var out_sumL = sumNumeric(getRangeValuesSafe(startRow, 12, dataRowCount, 1));
  var out_sumM = sumNumeric(getRangeValuesSafe(startRow, 13, dataRowCount, 1));
  var out_sumO = sumNumeric(getRangeValuesSafe(startRow, 15, dataRowCount, 1));
  var out_sumP = sumNumeric(getRangeValuesSafe(startRow, 16, dataRowCount, 1));
  var out_sumR = sumNumeric(getRangeValuesSafe(startRow, 18, dataRowCount, 1));

  // ----- Attendance % using H4 (working days) -----
  var workingDays = Number(sheet.getRange(4, 8).getValue()) || 0; // H4
  var out_att_percent = (workingDays > 0 && totalStudents > 0) 
                        ? (sumH / (workingDays * totalStudents)) * 100
                        : 0;

  // ----- Exam pass % helper (column letters AL, BB, BR) -----
  function passPercentFromLetter(colLetter) {
    var colNumber = sheet.getRange(colLetter + "1").getColumn();
    var vals = getRangeValuesSafe(startRow, colNumber, dataRowCount, 1);
    var passed = 0;
    for (var j = 0; j < vals.length; j++) {
      if ((vals[j][0] || "").toString().toUpperCase().trim() === "PASSED") passed++;
    }
    return totalStudents > 0 ? (passed / totalStudents) * 100 : 0;
  }

  var passAL = passPercentFromLetter("AL");
  var passBB = passPercentFromLetter("BB");
  var passBR = passPercentFromLetter("BR");

  // ----- Write results to swader -----
  swader.getRange(3, swaderCol).setValue(out_s_nozero);
  swader.getRange(4, swaderCol).setValue(out_s_withzero);
  swader.getRange(6, swaderCol).setValue(out_sumG);
  swader.getRange(7, swaderCol).setValue(Math.round(out_att_percent * 100) / 100);
  swader.getRange(8, swaderCol).setValue(out_sumL);
  swader.getRange(9, swaderCol).setValue(out_sumM);
  swader.getRange(10, swaderCol).setValue(out_sumO);
  swader.getRange(11, swaderCol).setValue(out_sumP);
  swader.getRange(12, swaderCol).setValue(out_sumR);
  swader.getRange(13, swaderCol).setValue(Math.round(passAL * 100) / 100);
  swader.getRange(14, swaderCol).setValue(Math.round(passBB * 100) / 100);
  swader.getRange(15, swaderCol).setValue(Math.round(passBR * 100) / 100);
}





// keep clearColumnG as separate function
function clearColumnG() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var allowedSheets = ["class1","class2","class3","class4","class5",
                       "class6","class7","class8","class9","class10",
                       "class11","class12"];
  
  allowedSheets.forEach(function(sheetName) {
    var sheet = ss.getSheetByName(sheetName);
    if (sheet) {
      var lastRow = sheet.getLastRow();
      if (lastRow >= 6) {
        sheet.getRange(6, 7, lastRow - 5, 1).clearContent();
      }
    }
  });
   
}


