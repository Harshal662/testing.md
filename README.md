Got it! If you are working exclusively with Excel files for both the input files, I'll modify the code accordingly to handle Excel files instead of using `CSVReader`. Here's the updated code:

---

### Updated Code for Excel Files Only

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.util.HashMap;
import java.util.Map;

public class UniqueCombinationCounterExcel {
    public static void main(String[] args) {
        String inputExcel1Path = "input1.xlsx"; // Path to the first input Excel
        String inputExcel2Path = "input2.xlsx"; // Path to the second input Excel
        String outputExcelPath = "output.xlsx"; // Path to the output Excel

        try {
            // Step 1: Count unique combinations from the first Excel file
            Map<String, Integer> combinationCount = new HashMap<>();

            try (FileInputStream fis = new FileInputStream(inputExcel1Path);
                 Workbook workbook = new XSSFWorkbook(fis)) {

                Sheet sheet = workbook.getSheetAt(0);
                int refCol = -1, currencyCol = -1;

                // Identify relevant columns from the header
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

                // Count unique combinations
                for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                    Row row = sheet.getRow(i);
                    if (row == null) continue;

                    String key = row.getCell(refCol).getStringCellValue() + "," +
                            row.getCell(currencyCol).getStringCellValue();
                    combinationCount.put(key, combinationCount.getOrDefault(key, 0) + 1);
                }
            }

            // Step 2: Process the second Excel file to count RESULT occurrences for each combination
            Map<String, Map<String, Integer>> resultCounts = new HashMap<>();

            try (FileInputStream fis = new FileInputStream(inputExcel2Path);
                 Workbook workbook = new XSSFWorkbook(fis)) {

                Sheet sheet = workbook.getSheetAt(0);
                int refCol = -1, currencyCol = -1, resultCol = -1;

                // Identify relevant columns from the header
                Row headerRow = sheet.getRow(0);
                for (int i = 0; i < headerRow.getPhysicalNumberOfCells(); i++) {
                    String header = headerRow.getCell(i).getStringCellValue();
                    if ("YOUR_REFERENCE".equalsIgnoreCase(header)) {
                        refCol = i;
                    } else if ("CURRENCY".equalsIgnoreCase(header)) {
                        currencyCol = i;
                    } else if ("RESULT".equalsIgnoreCase(header)) {
                        resultCol = i;
                    }
                }

                if (refCol == -1 || currencyCol == -1 || resultCol == -1) {
                    throw new IllegalArgumentException("Required columns not found in the second Excel.");
                }

                // Count RESULT occurrences for each combination
                for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                    Row row = sheet.getRow(i);
                    if (row == null) continue;

                    String combination = row.getCell(refCol).getStringCellValue() + "," +
                            row.getCell(currencyCol).getStringCellValue();
                    String result = row.getCell(resultCol).getStringCellValue();

                    resultCounts.putIfAbsent(combination, new HashMap<>());
                    Map<String, Integer> resultMap = resultCounts.get(combination);
                    resultMap.put(result, resultMap.getOrDefault(result, 0) + 1);
                }
            }

            // Step 3: Write results to the output Excel file
            try (Workbook workbook = new XSSFWorkbook()) {
                Sheet sheet = workbook.createSheet("Results");

                // Write header
                Row headerRow = sheet.createRow(0);
                headerRow.createCell(0).setCellValue("Combination");
                headerRow.createCell(1).setCellValue("Count");
                headerRow.createCell(2).setCellValue("Success");
                headerRow.createCell(3).setCellValue("Transaction Missing");
                headerRow.createCell(4).setCellValue("Incorrect Product Detected");

                // Write data rows
                int rowIndex = 1;
                for (Map.Entry<String, Integer> entry : combinationCount.entrySet()) {
                    String combination = entry.getKey();
                    int count = entry.getValue();

                    Map<String, Integer> resultMap = resultCounts.getOrDefault(combination, new HashMap<>());

                    Row row = sheet.createRow(rowIndex++);
                    row.createCell(0).setCellValue(combination);
                    row.createCell(1).setCellValue(count);
                    row.createCell(2).setCellValue(resultMap.getOrDefault("Success", 0));
                    row.createCell(3).setCellValue(resultMap.getOrDefault("Transaction Missing", 0));
                    row.createCell(4).setCellValue(resultMap.getOrDefault("Incorrect Product Detected", 0));
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
}
```

---

### Explanation

1. **Input Excel Files**:
   - The code reads the first Excel file (`input1.xlsx`) to count unique combinations of `YOUR_REFERENCE` and `ORIG_AMOUNT_CURRENCY`.
   - It reads the second Excel file (`input2.xlsx`) to map each combination to `RESULT` types (`Success`, `Transaction Missing`, and `Incorrect Product Detected`).

2. **Result Mapping**:
   - A `Map<String, Map<String, Integer>>` is used to store the counts of `RESULT` types for each unique combination.

3. **Output Excel**:
   - The output Excel file (`output.xlsx`) contains columns for `Combination`, `Count`, `Success`, `Transaction Missing`, and `Incorrect Product Detected`.
   - Each row corresponds to a unique combination with counts for all result types.

---

### Example Output (`output.xlsx`)

| Combination       | Count | Success | Transaction Missing | Incorrect Product Detected |
|--------------------|-------|---------|---------------------|----------------------------|
| REF1,USD          | 2     | 2       | 0                   | 0                          |
| REF1,EUR          | 1     | 0       | 1                   | 0                          |
| REF2,USD          | 2     | 0       | 1                   | 1                          |

This ensures the counts for each result type are correctly mapped to their combinations.

Let me know if further adjustments are required!
