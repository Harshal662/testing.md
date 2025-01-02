Here’s the corrected logic to achieve the specified output, ensuring that the counts for `Success`, `Transaction Missing`, and `Incorrect Product Detected` are displayed correctly alongside the combinations. Each combination will have its respective counts for the result types.

---

### Corrected Code

```java
import com.opencsv.CSVReader;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.util.HashMap;
import java.util.Map;

public class UniqueCombinationCounter {
    public static void main(String[] args) {
        String inputCsvPath = "input.csv"; // Path to input CSV
        String inputExcelPath = "reference.xlsx"; // Path to the second input Excel
        String outputExcelPath = "output.xlsx"; // Path to output Excel

        try {
            // Step 1: Count unique combinations from the CSV file
            Map<String, Integer> combinationCount = new HashMap<>();
            try (CSVReader csvReader = new CSVReader(new FileReader(inputCsvPath))) {
                String[] headers = csvReader.readNext(); // Read header row
                if (headers == null) {
                    throw new IllegalArgumentException("Input CSV is empty.");
                }

                int refIndex = -1;
                int currencyIndex = -1;

                // Identify the indexes for the desired columns
                for (int i = 0; i < headers.length; i++) {
                    if ("YOUR_REFERENCE".equalsIgnoreCase(headers[i])) {
                        refIndex = i;
                    } else if ("ORIG_AMOUNT_CURRENCY".equalsIgnoreCase(headers[i])) {
                        currencyIndex = i;
                    }
                }

                if (refIndex == -1 || currencyIndex == -1) {
                    throw new IllegalArgumentException("Required columns not found.");
                }

                String[] row;
                while ((row = csvReader.readNext()) != null) {
                    String key = row[refIndex] + "," + row[currencyIndex];
                    combinationCount.put(key, combinationCount.getOrDefault(key, 0) + 1);
                }
            }

            // Step 2: Process the second Excel file to map combinations to RESULT counts
            Map<String, Map<String, Integer>> resultCounts = new HashMap<>();

            try (FileInputStream fis = new FileInputStream(inputExcelPath);
                 Workbook workbook = new XSSFWorkbook(fis)) {

                Sheet sheet = workbook.getSheetAt(0);
                int refCol = -1, currencyCol = -1, resultCol = -1;

                // Identify the relevant columns from the header
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
                    throw new IllegalArgumentException("Required columns not found in reference Excel.");
                }

                // Iterate over rows and count RESULT occurrences for each combination
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

### Explanation of Changes
1. **Result Mapping**: Added a `Map<String, Map<String, Integer>>` to store counts of "Success," "Transaction Missing," and "Incorrect Product Detected" for each combination.
2. **Result Lookup**: For each combination, the corresponding `RESULT` counts are fetched from the map and added to the Excel sheet.
3. **Dynamic Default Values**: If a combination has no result for a particular type, a default count of `0` is used.

---

### Example Output

#### Output Excel (`output.xlsx`)

| Combination       | Count | Success | Transaction Missing | Incorrect Product Detected |
|--------------------|-------|---------|---------------------|----------------------------|
| REF1,USD          | 2     | 2       | 0                   | 0                          |
| REF1,EUR          | 1     | 0       | 1                   | 0                          |
| REF2,USD          | 2     | 0       | 1                   | 1                          |

This ensures the total count of each combination matches the counts of the result types.

---

Let me know if further refinements or clarifications are needed!
