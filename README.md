Understood! I'll remove the **Third Excel Count** column and directly calculate and include only the required result counts from the third Excel. Here's the updated code:

---

### Updated Code (No "Third Excel Count")
```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

public class UniqueCombinationCounterExcel {
    public static void main(String[] args) {
        String inputExcel1Path = "input1.xlsx"; // Path to the first input Excel
        String inputExcel2Path = "input2.xlsx"; // Path to the second input Excel
        String inputExcel3Path = "input3.xlsx"; // Path to the third input Excel
        String outputExcelPath = "output.xlsx"; // Path to the output Excel

        try {
            // Step 1: Count unique combinations from the first Excel file
            Map<String, Integer> combinationCount = processFirstExcel(inputExcel1Path);

            // Step 2: Map RESULT counts from the second Excel file
            Map<String, Map<String, Integer>> resultCounts1 = processSecondExcel(inputExcel2Path);

            // Step 3: Handle redundant rows and map results from the third Excel file
            Map<String, Map<String, Integer>> resultCounts3 = processThirdExcel(inputExcel3Path);

            // Step 4: Write results to the output Excel file
            writeResultsToExcel(outputExcelPath, combinationCount, resultCounts1, resultCounts3);

            System.out.println("Processing complete. Output saved to: " + outputExcelPath);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // Process the first Excel file
    private static Map<String, Integer> processFirstExcel(String inputExcel1Path) throws IOException {
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
        return combinationCount;
    }

    // Process the second Excel file
    private static Map<String, Map<String, Integer>> processSecondExcel(String inputExcel2Path) throws IOException {
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
        return resultCounts;
    }

    // Process the third Excel file
    private static Map<String, Map<String, Integer>> processThirdExcel(String inputExcel3Path) throws IOException {
        Map<String, Map<String, Integer>> resultCounts = new LinkedHashMap<>();
        try (FileInputStream fis = new FileInputStream(inputExcel3Path);
             Workbook workbook = new XSSFWorkbook(fis)) {

            Sheet sheet = workbook.getSheetAt(0);
            int billingCodeCol = -1, currencyCol = -1, resultCol = -1;

            // Identify relevant columns from the header
            Row headerRow = sheet.getRow(0);
            for (int i = 0; i < headerRow.getPhysicalNumberOfCells(); i++) {
                String header = headerRow.getCell(i).getStringCellValue();
                if ("BILLING_CODE".equalsIgnoreCase(header)) {
                    billingCodeCol = i;
                } else if ("CURRENCY".equalsIgnoreCase(header)) {
                    currencyCol = i;
                } else if ("RESULT".equalsIgnoreCase(header)) {
                    resultCol = i;
                }
            }

            if (billingCodeCol == -1 || currencyCol == -1 || resultCol == -1) {
                throw new IllegalArgumentException("Required columns not found in the third Excel.");
            }

            // Process rows and keep only the last occurrence of duplicates
            Map<String, Integer> rowIndexMap = new LinkedHashMap<>();
            for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                Row row = sheet.getRow(i);
                if (row == null) continue;

                String billingCode = row.getCell(billingCodeCol).getStringCellValue();
                rowIndexMap.put(billingCode, i); // Overwrite previous occurrences
            }

            // Count combinations using the last occurrence and count RESULTS
            for (int rowIndex : rowIndexMap.values()) {
                Row row = sheet.getRow(rowIndex);
                if (row == null) continue;

                String key = row.getCell(billingCodeCol).getStringCellValue() + "," +
                        row.getCell(currencyCol).getStringCellValue();
                String result = row.getCell(resultCol).getStringCellValue();

                resultCounts.putIfAbsent(key, new HashMap<>());
                Map<String, Integer> resultMap = resultCounts.get(key);
                resultMap.put(result, resultMap.getOrDefault(result, 0) + 1);
            }
        }
        return resultCounts;
    }

    // Write all results to the output Excel file
    private static void writeResultsToExcel(String outputExcelPath, Map<String, Integer> combinationCount1,
                                            Map<String, Map<String, Integer>> resultCounts1,
                                            Map<String, Map<String, Integer>> resultCounts3) throws IOException {

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
            for (Map.Entry<String, Integer> entry : combinationCount1.entrySet()) {
                String combination = entry.getKey();
                int count = entry.getValue();

                Map<String, Integer> resultMap1 = resultCounts1.getOrDefault(combination, new HashMap<>());
                Map<String, Integer> resultMap3 = resultCounts3.getOrDefault(combination, new HashMap<>());

                Row row = sheet.createRow(rowIndex++);
                row.createCell(0).setCellValue(combination);
                row.createCell(1).setCellValue(count);
                row.createCell(2).setCellValue(resultMap1.getOrDefault("Success", 0));
                row.createCell(3).setCellValue(resultMap1.getOrDefault("Transaction Missing", 0));
                row.createCell(4).setCellValue(resultMap1.getOrDefault("Incorrect Product Detected", 0));
                row.createCell(5).setCellValue(resultMap3.getOrDefault("Rules are not matching", 0));
                row.createCell(6).setCellValue(resultMap3.getOrDefault("Rules are overlapping and passing as per priority order", 0));
                row.createCell(7).setCellValue(resultMap3.getOrDefault("Rules are overlapping but not passing as per priority order", 0));
            }

            // Write to output file
            try (FileOutputStream fos = new FileOutputStream(outputExcelPath)) {
                workbook.write(fos);
            }
        }
    }
}
```

---

### Changes Made:
- **Removed Third Excel Count**: The column has been omitted from both the logic and the output file.
- The logic for processing the third Excel is retained and only calculates the counts for the three result types:
  - `"Rules are not matching"`
  - `"Rules are overlapping and passing as per priority order"`
  - `"Rules are overlapping but not passing as per priority order"`

This ensures the final Excel file includes only the required columns. Let me know if you need further refinements!
