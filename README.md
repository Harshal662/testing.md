Below is the edited code to convert **all Excel files** in a specified directory to either **CSV** or **Parquet** files. The program iterates through each `.xlsx` file in the directory, processes them, and generates corresponding output files.

---

### **Convert All Excel Files in a Directory to CSV**
#### **Code**
```java
import org.apache.poi.ss.usermodel.*;

import java.io.*;
import java.nio.file.*;
import java.util.List;

public class ConvertExcelToCSV {

    public static void main(String[] args) {
        String excelDirectoryPath = "excel_files"; // Directory containing Excel files
        String outputDirectoryPath = "output_csv"; // Directory to store CSV files

        try {
            // Ensure the output directory exists
            Files.createDirectories(Paths.get(outputDirectoryPath));

            // Get all .xlsx files from the input directory
            List<File> excelFiles = getExcelFiles(excelDirectoryPath);

            if (excelFiles.isEmpty()) {
                System.out.println("No Excel files found in the directory.");
                return;
            }

            // Convert each Excel file to CSV
            for (File excelFile : excelFiles) {
                System.out.println("Processing file: " + excelFile.getName());
                String csvFileName = excelFile.getName().replace(".xlsx", ".csv");
                String outputFilePath = Paths.get(outputDirectoryPath, csvFileName).toString();
                convertToCSV(excelFile, outputFilePath);
                System.out.println("Converted to CSV: " + outputFilePath);
            }
        } catch (IOException e) {
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

    // Method to convert an Excel file to CSV
    private static void convertToCSV(File excelFile, String outputFilePath) {
        try (FileInputStream fis = new FileInputStream(excelFile);
             Workbook workbook = WorkbookFactory.create(fis);
             PrintWriter writer = new PrintWriter(new FileWriter(outputFilePath))) {

            Sheet sheet = workbook.getSheetAt(0); // Convert only the first sheet
            for (Row row : sheet) {
                StringBuilder rowString = new StringBuilder();

                for (Cell cell : row) {
                    String cellValue = getCellValue(cell);
                    rowString.append(cellValue).append(","); // Separate values by commas
                }

                // Remove trailing comma and write the line
                if (rowString.length() > 0) {
                    rowString.setLength(rowString.length() - 1);
                }
                writer.println(rowString.toString());
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

---

### **Convert All Excel Files in a Directory to Parquet**
#### **Code**
```java
import org.apache.poi.ss.usermodel.*;
import org.apache.parquet.example.data.Group;
import org.apache.parquet.example.data.simple.SimpleGroupFactory;
import org.apache.parquet.hadoop.ParquetWriter;
import org.apache.parquet.hadoop.example.ExampleParquetWriter;
import org.apache.parquet.schema.MessageType;
import org.apache.parquet.schema.Types;

import java.io.File;
import java.io.FileInputStream;
import java.nio.file.*;
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
    private static void convertToParquet(File excelFile, String outputFilePath) {
        try (FileInputStream fis = new FileInputStream(excelFile);
             Workbook workbook = WorkbookFactory.create(fis)) {

            Sheet sheet = workbook.getSheetAt(0); // Convert only the first sheet

            // Define a Parquet schema
            MessageType schema = Types.buildMessage()
                    .required(Types.primitive(org.apache.parquet.schema.PrimitiveType.PrimitiveTypeName.BINARY)
                            .as(org.apache.parquet.schema.OriginalType.UTF8)
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

---

### **How It Works**
1. **Directory Handling**:
   - The program scans the directory for `.xlsx` files.
   - It ensures the output directory exists.

2. **File Conversion**:
   - Converts each `.xlsx` file to CSV or Parquet using the appropriate logic.

3. **Output Files**:
   - The output files are named the same as the input Excel files but with `.csv` or `.parquet` extensions.

---

### **Directory Structure**
```
project_directory/
├── excel_files/          # Input directory
│   ├── file1.xlsx
│   ├── file2.xlsx
│
├── output_csv/           # CSV output directory
│   ├── file1.csv
│   ├── file2.csv
│
├── output_parquet/       # Parquet output directory
│   ├── file1.parquet
│   ├── file2.parquet
```

Let me know if you need help with further customizations! 😊
