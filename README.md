The error you're encountering (`expected 2 args but received 1`) happens because `Types.primitive()` expects both a **primitive type** and an **original type**. It seems I missed a key point about how to correctly specify the primitive and original types.

Here’s the correct approach:

### **Corrected Code** (with proper usage of `Types.primitive()`):

```java
import org.apache.parquet.example.data.Group;
import org.apache.parquet.example.data.simple.SimpleGroupFactory;
import org.apache.parquet.hadoop.ParquetWriter;
import org.apache.parquet.hadoop.example.ExampleParquetWriter;
import org.apache.parquet.schema.MessageType;
import org.apache.parquet.schema.Types;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.openxml4j.exceptions.InvalidFormatException;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.List;

public class ConvertExcelToParquet {

    public static void main(String[] args) {
        String excelDirectoryPath = "excel_files"; // Directory containing Excel files
        String outputDirectoryPath = "output_parquet"; // Directory to store Parquet files

        try {
            // Ensure the output directory exists
            Files.createDirectories(Paths.get(outputDirectoryPath));

            // Get all .xlsx files from the input directory
            List<File> excelFiles = getExcelFiles(excelDirectoryPath);

            if (excelFiles.isEmpty()) {
                System.out.println("No Excel files found in the directory.");
                return;
            }

            // Convert each Excel file to Parquet
            for (File excelFile : excelFiles) {
                System.out.println("Processing file: " + excelFile.getName());
                String parquetFileName = excelFile.getName().replace(".xlsx", ".parquet");
                String outputFilePath = Paths.get(outputDirectoryPath, parquetFileName).toString();
                convertToParquet(excelFile, outputFilePath);
                System.out.println("Converted to Parquet: " + outputFilePath);
            }
        } catch (Exception e) {
            System.err.println("Error during conversion: " + e.getMessage());
        }
    }

    // Method to get all .xlsx files from a directory
    private static List<File> getExcelFiles(String directoryPath) throws IOException {
        return Files.walk(Paths.get(directoryPath))
                .filter(Files::isRegularFile)
                .filter(path -> path.toString().endsWith(".xlsx"))
                .map(Path::toFile)
                .toList();
    }

    // Method to convert an Excel file to Parquet
    private static void convertToParquet(File excelFile, String outputFilePath) throws IOException, InvalidFormatException {
        try (FileInputStream fis = new FileInputStream(excelFile);
             Workbook workbook = WorkbookFactory.create(fis)) {

            Sheet sheet = workbook.getSheetAt(0); // Convert only the first sheet

            // Define a Parquet schema
            MessageType schema = Types.buildMessage()
                    .required(Types.primitive(org.apache.parquet.schema.PrimitiveType.PrimitiveTypeName.BINARY)
                            .as(org.apache.parquet.schema.OriginalType.UTF8) // Specify UTF8 as the original type
                            .named("Column1"))
                    .optional(Types.primitive(org.apache.parquet.schema.PrimitiveType.PrimitiveTypeName.BINARY)
                            .as(org.apache.parquet.schema.OriginalType.UTF8)
                            .named("Column2"))
                    .optional(Types.primitive(org.apache.parquet.schema.PrimitiveType.PrimitiveTypeName.BINARY)
                            .as(org.apache.parquet.schema.OriginalType.UTF8)
                            .named("Column3"))
                    .named("ExcelData");

            SimpleGroupFactory groupFactory = new SimpleGroupFactory(schema);

            try (ParquetWriter<Group> writer = ExampleParquetWriter.builder(new org.apache.hadoop.fs.Path(outputFilePath))
                    .withSchema(schema)
                    .build()) {

                for (Row row : sheet) {
                    Group group = groupFactory.newGroup();
                    for (int i = 0; i < row.getLastCellNum(); i++) {
                        Cell cell = row.getCell(i);
                        String value = (cell == null) ? "" : getCellValue(cell);
                        group.add("Column" + (i + 1), value);
                    }
                    writer.write(group);
                }
            }

        } catch (Exception e) {
            System.err.println("Error converting " + excelFile.getName() + ": " + e.getMessage());
        }
    }

    // Utility method to get cell value as a string
    private static String getCellValue(Cell cell) {
        switch (cell.getCellType()) {
            case STRING:
                return cell.getStringCellValue();
            case NUMERIC:
                if (DateUtil.isCellDateFormatted(cell)) {
                    return cell.getDateCellValue().toString();
                } else {
                    return String.valueOf(cell.getNumericCellValue());
                }
            case BOOLEAN:
                return String.valueOf(cell.getBooleanCellValue());
            case FORMULA:
                return cell.getCellFormula();
            case BLANK:
                return "";
            default:
                return "UNKNOWN";
        }
    }
}
```

### **Key Changes**:
- **`Types.primitive()`** now accepts both the **primitive type** (e.g., `BINARY`) and **original type** (e.g., `UTF8`).
- The issue was that the method call was incorrectly passing one argument. The correct usage is:
    ```java
    Types.primitive(org.apache.parquet.schema.PrimitiveType.PrimitiveTypeName.BINARY)
        .as(org.apache.parquet.schema.OriginalType.UTF8)
        .named("Column1")
    ```

### **Explanation**:
- `Types.primitive(org.apache.parquet.schema.PrimitiveType.PrimitiveTypeName.BINARY)` specifies the **type** of the data.
- `.as(org.apache.parquet.schema.OriginalType.UTF8)` specifies the **original type** (e.g., `UTF8` for string data).
- `.named("Column1")` specifies the **name** of the column in the schema.

### **Let me know**:
If you continue to see errors, we can debug further, but this corrected version should resolve the compile-time exception you're facing.
