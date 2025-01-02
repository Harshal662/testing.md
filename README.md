Here’s the Java Maven code to achieve your requirement. The code reads an Excel file with two sheets, matches the `BillingCode` column, and adds the `PriorityOrder` column from the second sheet to the first sheet, handling multiple occurrences of the same `BillingCode` in the first sheet.

We will use the Apache POI library for working with Excel files.

### Maven Dependency
Add the following dependency to your `pom.xml`:
```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.3</version>
</dependency>
```

### Java Code
```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class ExcelUpdater {

    public static void main(String[] args) throws IOException {
        // Path to the Excel file
        String filePath = "path_to_your_excel_file.xlsx";

        // Open the Excel file
        FileInputStream fileInputStream = new FileInputStream(filePath);
        Workbook workbook = new XSSFWorkbook(fileInputStream);

        // Read the two sheets
        Sheet sheet1 = workbook.getSheetAt(0);
        Sheet sheet2 = workbook.getSheetAt(1);

        // Map to store BillingCode -> PriorityOrder
        Map<String, String> priorityOrderMap = new HashMap<>();

        // Assuming the headers are in the first row
        int billingCodeIndexSheet2 = -1;
        int priorityOrderIndex = -1;

        // Get the indexes for BillingCode and PriorityOrder in Sheet 2
        Row headerRowSheet2 = sheet2.getRow(0);
        for (Cell cell : headerRowSheet2) {
            if ("BillingCode".equalsIgnoreCase(cell.getStringCellValue())) {
                billingCodeIndexSheet2 = cell.getColumnIndex();
            } else if ("PriorityOrder".equalsIgnoreCase(cell.getStringCellValue())) {
                priorityOrderIndex = cell.getColumnIndex();
            }
        }

        // Populate the map from Sheet 2
        for (Row row : sheet2) {
            if (row.getRowNum() == 0) continue; // Skip header row
            String billingCode = row.getCell(billingCodeIndexSheet2).getStringCellValue();
            String priorityOrder = row.getCell(priorityOrderIndex).getStringCellValue();
            priorityOrderMap.put(billingCode, priorityOrder);
        }

        // Add PriorityOrder column to Sheet 1
        int billingCodeIndexSheet1 = -1;
        int newColumnIndex = -1;

        // Get the BillingCode index in Sheet 1
        Row headerRowSheet1 = sheet1.getRow(0);
        for (Cell cell : headerRowSheet1) {
            if ("BillingCode".equalsIgnoreCase(cell.getStringCellValue())) {
                billingCodeIndexSheet1 = cell.getColumnIndex();
            }
        }

        // Add the new column header
        newColumnIndex = headerRowSheet1.getLastCellNum();
        headerRowSheet1.createCell(newColumnIndex).setCellValue("PriorityOrder");

        // Update rows with PriorityOrder from the map
        for (Row row : sheet1) {
            if (row.getRowNum() == 0) continue; // Skip header row
            String billingCode = row.getCell(billingCodeIndexSheet1).getStringCellValue();
            String priorityOrder = priorityOrderMap.getOrDefault(billingCode, ""); // Default to empty if not found
            row.createCell(newColumnIndex).setCellValue(priorityOrder);
        }

        // Write back to the file
        FileOutputStream fileOutputStream = new FileOutputStream(filePath);
        workbook.write(fileOutputStream);

        // Close resources
        fileOutputStream.close();
        workbook.close();
        fileInputStream.close();

        System.out.println("Excel file updated successfully!");
    }
}
```

### Key Points:
1. Replace `"path_to_your_excel_file.xlsx"` with the path to your Excel file.
2. The program reads both sheets, maps `BillingCode` to `PriorityOrder`, and appends the `PriorityOrder` column to the first sheet.
3. It handles multiple occurrences of `BillingCode` in the first sheet.

Let me know if you need any further clarifications!
