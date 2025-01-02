To include the new logic, we'll incorporate a third Excel file with columns `BILLING_CODE`, `CURRENCY`, and `RESULT`. We'll count the occurrences of the three new result types (`Rules are not matching`, `Rules are overlapping and passing as per priority order`, and `Rules are overlapping but not passing as per priority order`) for each combination and add these counts to the final output.

---

### Updated Code

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
        String inputExcel3Path = "input3.xlsx"; // Path to the third input Excel
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

            // Step 2: Process the second Excel file for RESULT counts
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

            // Step 3: Process the third Excel file for new RESULT counts
            Map<String, Map<String, Integer>> newResultCounts = new HashMap<>();

            try (FileInputStream fis = new FileInputStream(inputExcel3Path);
                 Workbook workbook = new XSSFWorkbook(fis)) {

                Sheet sheet = workbook.getSheetAt(0);
                int refCol = -1, currencyCol = -1, resultCol = -1;

                // Identify relevant columns from the header
                Row headerRow = sheet.getRow(0);
                for (int i = 0; i < headerRow.getPhysicalNumberOfCells(); i++) {
                    String header = headerRow.getCell(i).getStringCellValue();
                    if ("BILLING_CODE".equalsIgnoreCase(header)) {
                        refCol = i;
                    } else if ("CURRENCY".equalsIgnoreCase(header)) {
                        currencyCol = i;
                    } else if ("RESULT".equalsIgnoreCase(header)) {
                        resultCol = i;
                    }
                }

                if (refCol == -1 || currencyCol == -1 || resultCol == -1) {
                    throw new IllegalArgumentException("Required columns not found in the third Excel.");
                }

                // Count RESULT occurrences for each combination
                for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                    Row row = sheet.getRow(i);
                    if (row == null) continue;

                    String combination = row.getCell(refCol).getStringCellValue() + "," +
                            row.getCell(currencyCol).getStringCellValue();
                    String result = row.getCell(resultCol).getStringCellValue();

                    newResultCounts.putIfAbsent(combination, new HashMap<>());
                    Map<String, Integer> resultMap = newResultCounts.get(combination);
                    resultMap.put(result, resultMap.getOrDefault(result, 0) + 1);
                }
            }

            // Step 4: Write results to the output Excel file
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

                    Map<String, Integer> resultMap = resultCounts.getOrDefault(combination, new HashMap<>());
                    Map<String, Integer> newResultMap = newResultCounts.getOrDefault(combination, new HashMap<>());

                    Row row = sheet.createRow(rowIndex++);
                    row.createCell(0).setCellValue(combination);
                    row.createCell(1).setCellValue(count);
                    row.createCell(2).setCellValue(resultMap.getOrDefault("Success", 0));
                    row.createCell(3).setCellValue(resultMap.getOrDefault("Transaction Missing", 0));
                    row.createCell(4).setCellValue(resultMap.getOrDefault("Incorrect Product Detected", 0));
                    row.createCell(5).setCellValue(newResultMap.getOrDefault("Rules are not matching", 0));
                    row.createCell(6).setCellValue(newResultMap.getOrDefault("Rules are overlapping and passing as per priority order", 0));
                    row.createCell(7).setCellValue(newResultMap.getOrDefault("Rules are overlapping but not passing as per priority order", 0));
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

1. **Input Files**:
   - `input1.xlsx`: Contains `YOUR_REFERENCE` and `ORIG_AMOUNT_CURRENCY` for counting unique combinations.
   - `input2.xlsx`: Maps combinations to `Success`, `Transaction Missing`, and `Incorrect Product Detected`.
   - `input3.xlsx`: Maps combinations to three new result types.

2. **New Result Mapping**:
   - Added a separate map (`newResultCounts`) for the new result types:  
     - `Rules are not matching`
     - `Rules are overlapping and passing as per priority order`
     - `Rules are overlapping but not passing as per priority order`.

3. **Output File**:
   - The output Excel contains columns for all result types, ensuring the counts are displayed for each combination.

---

### Example Output

#### Output Excel (`output.xlsx`)

| Combination       | Count | Success | Transaction Missing | Incorrect Product Detected | Rules are not matching | Rules are overlapping and passing as per priority order | Rules are overlapping but not passing as per priority order |
|--------------------|-------|---------|---------------------|----------------------------|-------------------------|----------------------------------------------------------|------------------------------------------------------------|
| REF1,USD          | 2     | 2       | 0                   | 0                          | 1                       | 0                                                        | 1                                                          |
| REF1,EUR          | 1     | 0       | 1                   | 0                          | 0                       | 1                                                        | 0                                                          |
| REF2,USD          | 2     | 0       | 1                   | 1                          | 0                       | 1                                                        | 1                                                          |

This structure ensures all result types are accounted for across the combinations. Let me know if any further refinements are needed!
