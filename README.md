To implement the solution you are asking for, we will break it down into a few steps:

1. **Read the first Excel file and filter rows based on the "RESULT" column.**
2. **Read the second Excel file and match rows based on the `LOCAL_REFERENCE_NO`.**
3. **Store the data from the first Excel file in a map using `YOUR_REFERENCE` and `INITIAL_PRICE_ITEM_CD` as keys.**
4. **Compare the data between the two files based on `LOCAL_REFERENCE_NO`.**
5. **Perform the comparison twice to ensure the data matches correctly.**

Let's start by reading the Excel files, storing data, and performing comparisons in Java.

### 1. **Add Maven Dependencies**

You will need the Apache POI library to read Excel files. Add this to your `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version>5.2.3</version>
    </dependency>
</dependencies>
```

### 2. **Code to Read the Files and Perform the Required Actions**

The following code does the following:

- Reads the first Excel file and filters rows where `RESULT` is `"Incorrect Product Detected"`.
- Reads the second Excel file and matches rows based on `LOCAL_REFERENCE_NO`.
- Stores the data from the first file in a `Map<String, Map<String, String>>`, where the keys are `YOUR_REFERENCE` and `INITIAL_PRICE_ITEM_CD`.
- Matches and compares the data between both files.

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.*;

public class ExcelComparison {

    public static void main(String[] args) {
        String filePath1 = "path_to_your_first_excel_file.xlsx"; // First file path
        String filePath2 = "path_to_your_second_excel_file.xlsx"; // Second file path

        Map<String, Map<String, String>> dataMapFromFile1 = new HashMap<>();
        Map<String, Map<String, String>> dataMapFromFile2 = new HashMap<>();

        try {
            // Read first Excel file and filter rows based on RESULT column
            readFirstFile(filePath1, dataMapFromFile1);
            // Read second Excel file and match LOCAL_REFERENCE_NO
            readSecondFile(filePath2, dataMapFromFile2);
            
            // Perform the comparison between both maps
            compareData(dataMapFromFile1, dataMapFromFile2);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // Method to read the first Excel file
    private static void readFirstFile(String filePath, Map<String, Map<String, String>> dataMap) throws IOException {
        try (FileInputStream fis = new FileInputStream(filePath);
             Workbook workbook = new XSSFWorkbook(fis)) {

            Sheet sheet = workbook.getSheetAt(0);
            Row headerRow = sheet.getRow(0);
            int resultColumnIndex = -1, yourReferenceIndex = -1, initialPriceItemCdIndex = -1;

            List<String> columnNames = new ArrayList<>();
            for (Cell cell : headerRow) {
                String columnName = cell.getStringCellValue();
                columnNames.add(columnName);

                if ("RESULT".equalsIgnoreCase(columnName)) resultColumnIndex = cell.getColumnIndex();
                if ("YOUR_REFERENCE".equalsIgnoreCase(columnName)) yourReferenceIndex = cell.getColumnIndex();
                if ("INITIAL_PRICE_ITEM_CD".equalsIgnoreCase(columnName)) initialPriceItemCdIndex = cell.getColumnIndex();
            }

            if (resultColumnIndex == -1 || yourReferenceIndex == -1 || initialPriceItemCdIndex == -1) {
                throw new IllegalArgumentException("Required columns are missing in the first Excel file.");
            }

            for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                Row row = sheet.getRow(i);
                if (row == null) continue;

                Cell resultCell = row.getCell(resultColumnIndex);
                if (resultCell != null && "Incorrect Product Detected".equalsIgnoreCase(resultCell.toString())) {
                    String yourReference = row.getCell(yourReferenceIndex).toString();
                    String initialPriceItemCd = row.getCell(initialPriceItemCdIndex).toString();

                    Map<String, String> rowData = new HashMap<>();
                    for (int j = 0; j < columnNames.size(); j++) {
                        if (j == resultColumnIndex || j == yourReferenceIndex || j == initialPriceItemCdIndex) continue;
                        rowData.put(columnNames.get(j), row.getCell(j) != null ? row.getCell(j).toString() : "");
                    }

                    // Combine YOUR_REFERENCE and INITIAL_PRICE_ITEM_CD to form the key
                    String key = yourReference + "_" + initialPriceItemCd;
                    dataMap.put(key, rowData);
                }
            }
        }
    }

    // Method to read the second Excel file and match LOCAL_REFERENCE_NO
    private static void readSecondFile(String filePath, Map<String, Map<String, String>> dataMap) throws IOException {
        try (FileInputStream fis = new FileInputStream(filePath);
             Workbook workbook = new XSSFWorkbook(fis)) {

            Sheet sheet = workbook.getSheetAt(0);
            Row headerRow = sheet.getRow(0);
            int localReferenceNoIndex = -1;

            List<String> columnNames = new ArrayList<>();
            for (Cell cell : headerRow) {
                String columnName = cell.getStringCellValue();
                columnNames.add(columnName);

                if ("LOCAL_REFERENCE_NO".equalsIgnoreCase(columnName)) {
                    localReferenceNoIndex = cell.getColumnIndex();
                }
            }

            if (localReferenceNoIndex == -1) {
                throw new IllegalArgumentException("LOCAL_REFERENCE_NO column is missing in the second Excel file.");
            }

            for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                Row row = sheet.getRow(i);
                if (row == null) continue;

                String localReferenceNo = row.getCell(localReferenceNoIndex).toString();
                Map<String, String> rowData = new HashMap<>();
                for (int j = 0; j < columnNames.size(); j++) {
                    rowData.put(columnNames.get(j), row.getCell(j) != null ? row.getCell(j).toString() : "");
                }

                dataMap.put(localReferenceNo, rowData);
            }
        }
    }

    // Method to compare the data between both maps
    private static void compareData(Map<String, Map<String, String>> dataMap1, Map<String, Map<String, String>> dataMap2) {
        for (Map.Entry<String, Map<String, String>> entry1 : dataMap1.entrySet()) {
            String key1 = entry1.getKey();
            Map<String, String> values1 = entry1.getValue();

            for (Map.Entry<String, Map<String, String>> entry2 : dataMap2.entrySet()) {
                String key2 = entry2.getKey();
                if (!key1.equals(key2)) continue; // Match LOCAL_REFERENCE_NO

                Map<String, String> values2 = entry2.getValue();

                // Compare the two datasets for each column
                for (Map.Entry<String, String> column1 : values1.entrySet()) {
                    String columnName = column1.getKey();
                    String value1 = column1.getValue();
                    String value2 = values2.getOrDefault(columnName, "");

                    if (!value1.equals(value2)) {
                        System.out.println("Mismatch in column '" + columnName + "' for key: " + key1);
                        System.out.println("File 1: " + value1);
                        System.out.println("File 2: " + value2);
                    }
                }
            }
        }
    }
}
```

### Key Steps in the Code:
1. **Reading the First Excel File:**
   - Filters the rows based on the "RESULT" column to only include rows where `"Incorrect Product Detected"` is written.
   - Stores the relevant data in `dataMapFromFile1`, using a composite key of `YOUR_REFERENCE` and `INITIAL_PRICE_ITEM_CD`.

2. **Reading the Second Excel File:**
   - Matches rows based on the `LOCAL_REFERENCE_NO` from the second Excel file.
   - Stores data in `dataMapFromFile2`, where each entry is mapped by `LOCAL_REFERENCE_NO`.

3. **Comparing Data:**
   - For each key in `dataMapFromFile1`, it compares the corresponding data in `dataMapFromFile2` for matching `LOCAL_REFERENCE_NO`.
   - If any data mismatch is found, it is printed out.

### How to Use:
1. Replace `"path_to_your_first_excel_file.xlsx"` and `"path_to_your_second_excel_file.xlsx"` with the actual paths to your Excel files.
2. Run the program to filter, compare, and print mismatched data.

This solution compares the data from the two files, ensuring that you only compare the rows where `"Incorrect Product Detected"` is mentioned in the first file, based on the `LOCAL_REFERENCE_NO`.
