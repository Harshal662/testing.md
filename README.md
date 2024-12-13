Here is a Java Maven code snippet to read an Excel file and store the data in a `Map<String, List<Map<String, String>>>` structure, where the key is the `BILING_CODE` and the values are lists of maps containing the rest of the column data. 

You can use the Apache POI library to handle Excel files. Ensure you include the Apache POI dependency in your Maven `pom.xml`.

### Maven Dependency
Add this to your `pom.xml`:
```xml
<dependencies>
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version>5.2.3</version>
    </dependency>
</dependencies>
```

### Java Code
```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.*;

public class ExcelToMap {

    public static void main(String[] args) {
        String filePath = "path_to_your_excel_file.xlsx"; // Replace with your Excel file path
        Map<String, List<Map<String, String>>> dataMap = new HashMap<>();

        try (FileInputStream fis = new FileInputStream(filePath);
             Workbook workbook = new XSSFWorkbook(fis)) {

            Sheet sheet = workbook.getSheetAt(0);
            Row headerRow = sheet.getRow(0);
            int billingCodeIndex = -1;
            List<String> columnNames = new ArrayList<>();

            // Identify columns and find the index of BILING_CODE
            for (Cell cell : headerRow) {
                String columnName = cell.getStringCellValue();
                columnNames.add(columnName);
                if (columnName.equalsIgnoreCase("BILING_CODE")) {
                    billingCodeIndex = cell.getColumnIndex();
                }
            }

            if (billingCodeIndex == -1) {
                throw new IllegalArgumentException("BILING_CODE column not found in the Excel sheet.");
            }

            // Iterate through rows and populate the map
            for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                Row row = sheet.getRow(i);
                if (row == null) continue;

                String billingCode = row.getCell(billingCodeIndex).getStringCellValue();
                Map<String, String> rowData = new HashMap<>();

                for (int j = 0; j < row.getLastCellNum(); j++) {
                    if (j == billingCodeIndex) continue; // Skip the key column
                    Cell cell = row.getCell(j);
                    String cellValue = cell != null ? cell.toString() : "";
                    rowData.put(columnNames.get(j), cellValue);
                }

                dataMap.computeIfAbsent(billingCode, k -> new ArrayList<>()).add(rowData);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }

        // Print the map
        for (Map.Entry<String, List<Map<String, String>>> entry : dataMap.entrySet()) {
            System.out.println("BILING_CODE: " + entry.getKey());
            for (Map<String, String> values : entry.getValue()) {
                System.out.println(values);
            }
            System.out.println();
        }
    }
}
```

### Explanation
1. **Dependencies**: The code uses Apache POI to read Excel files.
2. **Map Structure**: The map is structured as:
   - Key: `BILING_CODE`
   - Value: A list of maps containing the remaining column names and their respective values.
3. **Logic**:
   - The first row is treated as the header to get column names.
   - Data rows are read and stored as key-value pairs in the nested map.
   - If duplicate `BILING_CODE` values are found, their data is appended to the list of values.
4. **Print Output**: Iterates over the map and prints the content for debugging.

Replace `"path_to_your_excel_file.xlsx"` with the actual path to your Excel file, and this code should work seamlessly.
