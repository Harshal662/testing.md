Here is the updated Java code to handle the additional requirements:

1. **Handling Redundant Rows**: For the "YOUR_REFERENCE" column, only the last occurrence of duplicates is retained. The duplicates are one after another.
2. **Processing Additional Excel**: A new Excel file is processed containing columns "BILLING_CODE," "CURRENCY," and "RESULT." This file counts the values for each result type, similar to the previous logic.
3. **Combining Logic**: The pairing of "BILLING_CODE" and "CURRENCY" is handled, and counts are calculated.

---

### Updated Code

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.util.*;

public class AdvancedCombinationCounter {
    public static void main(String[] args) {
        String inputExcel1Path = "input1.xlsx"; // First Excel
        String inputExcel2Path = "input2.xlsx"; // Second Excel
        String inputExcel3Path = "input3.xlsx"; // Third Excel (with BILLING_CODE, etc.)
        String outputExcelPath = "output.xlsx"; // Output Excel

        try {
            // Step 1: Handle redundant rows and count unique combinations from the first Excel file
            Map<String, Integer> combinationCount = new HashMap<>();
            List<String> uniqueRows = new ArrayList<>();

            try (FileInputStream fis = new FileInputStream(inputExcel1Path);
                 Workbook workbook = new XSSFWorkbook(fis)) {

                Sheet sheet = workbook.getSheetAt(0);
                int refCol = -1, currencyCol = -1;

                // Identify relevant columns
                Row headerRow = sheet.getRow(0);
                for (int i = 0; i < headerRow.getPhysicalNumberOfCells(); i++) {
                    String header = headerRow.getCell(i).getStringCellValue();
                    if ("YOUR_REFERENCE".equalsIgnoreCase(header)) {
                        refCol = i;
                    } else if ("ORIG_AMOUNT_CURRENCY".equalsIgnoreCase(header)) {
                        currencyCol = i;
                    }
                }

                if (refCol == -1 || currencyCol == -1) {
                    throw new IllegalArgumentException("Required columns not found in the first Excel.");
                }

                // Process rows to retain only the last occurrence of duplicates
                String lastValue = null;
                for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                    Row row = sheet.getRow(i);
                    if (row == null) continue;

                    String refValue = row.getCell(refCol).getStringCellValue();
                    String currencyValue = row.getCell(currencyCol).getStringCellValue();
                    String key = refValue + "," + currencyValue;

                    // Detect duplicates and keep only the last occurrence
                    if (!key.equals(lastValue)) {
                        uniqueRows.add(key);
                    }
                    lastValue = key;
                }

                // Count combinations
                for (String row : uniqueRows) {
                    combinationCount.put(row, combinationCount.getOrDefault(row, 0) + 1);
                }
            }

            // Step 2: Process the second Excel file for additional result counts
            Map<String, Map<String, Integer>> resultCounts1 = new HashMap<>();
            Map<String, Map<String, Integer>> resultCounts2 = new HashMap<>();

            // Process second Excel
            processResultExcel(inputExcel2Path, resultCounts1, "YOUR_REFERENCE", "CURRENCY", "RESULT");
            // Process third Excel
            processResultExcel(inputExcel3Path, resultCounts2, "BILLING_CODE", "CURRENCY", "RESULT");

            // Step 3: Write results to the output Excel
            try (Workbook workbook = new XSSFWorkbook()) {
                Sheet sheet = workbook.createSheet("Results");

                // Write header
                Row headerRow = sheet.createRow(0);
                headerRow.createCell(0).setCellValue("Combination");
                headerRow.createCell(1).setCellValue("Count");
                headerRow.createCell(2).setCellValue("Success");
                headerRow.createCell(3).setCellValue("Transaction Missing");
                headerRow.createCell(4).setCellValue("Incorrect Product Detected");
                headerRow.createCell(5).setCellValue("Rules are not matching");
                headerRow.createCell(6).setCellValue("Rules are overlapping and passing as per priority order");
                headerRow.createCell(7).setCellValue("Rules are overlapping but not passing as per priority order");

                // Write data rows
                int rowIndex = 1;
                for (Map.Entry<String, Integer> entry : combinationCount.entrySet()) {
                    String combination = entry.getKey();
                    int count = entry.getValue();

                    Map<String, Integer> resultMap1 = resultCounts1.getOrDefault(combination, new HashMap<>());
                    Map<String, Integer> resultMap2 = resultCounts2.getOrDefault(combination, new HashMap<>());

                    Row row = sheet.createRow(rowIndex++);
                    row.createCell(0).setCellValue(combination);
                    row.createCell(1).setCellValue(count);
                    row.createCell(2).setCellValue(resultMap1.getOrDefault("Success", 0));
                    row.createCell(3).setCellValue(resultMap1.getOrDefault("Transaction Missing", 0));
                    row.createCell(4).setCellValue(resultMap1.getOrDefault("Incorrect Product Detected", 0));
                    row.createCell(5).setCellValue(resultMap2.getOrDefault("Rules are not matching", 0));
                    row.createCell(6).setCellValue(resultMap2.getOrDefault("Rules are overlapping and passing as per priority order", 0));
                    row.createCell(7).setCellValue(resultMap2.getOrDefault("Rules are overlapping but not passing as per priority order", 0));
                }

                // Save to file
                try (FileOutputStream fileOut = new FileOutputStream(outputExcelPath)) {
                    workbook.write(fileOut);
                }
            }

            System.out.println("Processing complete. Output saved to: " + outputExcelPath);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // Helper method to process result Excel
    private static void processResultExcel(String excelPath, Map<String, Map<String, Integer>> resultCounts,
                                           String keyCol1, String keyCol2, String resultColName) throws IOException {
        try (FileInputStream fis = new FileInputStream(excelPath);
             Workbook workbook = new XSSFWorkbook(fis)) {

            Sheet sheet = workbook.getSheetAt(0);
            int keyCol1Index = -1, keyCol2Index = -1, resultColIndex = -1;

            // Identify relevant columns
            Row headerRow = sheet.getRow(0);
            for (int i = 0; i < headerRow.getPhysicalNumberOfCells(); i++) {
                String header = headerRow.getCell(i).getStringCellValue();
                if (keyCol1.equalsIgnoreCase(header)) {
                    keyCol1Index = i;
                } else if (keyCol2.equalsIgnoreCase(header)) {
                    keyCol2Index = i;
                } else if (resultColName.equalsIgnoreCase(header)) {
                    resultColIndex = i;
                }
            }

            if (keyCol1Index == -1 || keyCol2Index == -1 || resultColIndex == -1) {
                throw new IllegalArgumentException("Required columns not found in Excel: " + excelPath);
            }

            // Count RESULT occurrences for each combination
            for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                Row row = sheet.getRow(i);
                if (row == null) continue;

                String combination = row.getCell(keyCol1Index).getStringCellValue() + "," +
                        row.getCell(keyCol2Index).getStringCellValue();
                String result = row.getCell(resultColIndex).getStringCellValue();

                resultCounts.putIfAbsent(combination, new HashMap<>());
                Map<String, Integer> resultMap = resultCounts.get(combination);
                resultMap.put(result, resultMap.getOrDefault(result, 0) + 1);
            }
        }
    }
}
```

---

### Explanation of Changes

1. **Redundant Rows**:
   - In the first Excel, redundant rows are processed, and only the last occurrence of duplicate rows for `YOUR_REFERENCE` is retained.

2. **Additional Logic**:
   - The third Excel file (`input3.xlsx`) is processed for combinations of `BILLING_CODE` and `CURRENCY`, along with result counts.

3. **Output**:
   - The final Excel includes all the required columns with counts for both sets of results.

---

### Example Input and Output

#### Input Excel 1 (`input1.xlsx`)
| YOUR_REFERENCE | ORIG_AMOUNT_CURRENCY |
|----------------|-----------------------|
| REF1           | USD                  |
| REF1           | USD                  |
| REF2           | EUR                  |

#### Input Excel 2 (`input2.xlsx`)
| YOUR_REFERENCE | CURRENCY | RESULT             |
|----------------|----------|--------------------|
| REF1           | USD      | Success            |
| REF2           | EUR      | Transaction Missing|

#### Input Excel 3 (`input3.xlsx`)
| BILLING_CODE | CURRENCY | RESULT                                      |
|--------------|----------|---------------------------------------------|
| REF1         | USD      | Rules are overlapping but not passing...    |

#### Output Excel (`output.xlsx`)
| Combination   | Count | Success | Transaction Missing | Incorrect Product Detected | Rules are not matching | Rules are overlapping and passing | Rules are overlapping but not passing |
|---------------|-------|---------|---------------------|----------------------------|-------------------------|------------------------------------|----------------------------------------|
| REF1,USD      | 1     | 1       | 0                   | 0                          | 0                       | 0                                  | 1                                      |
| REF2,EUR      | 1     | 0       | 1                   | 0                          | 0                       | 0                                  | 0                                      |

---

Let me know if further refinements are needed!
