Here's an example of a Java program to **read multiple Excel files**, where you can later modify it to perform specific operations on the data. This code uses **Apache POI** to read `.xlsx` files.

---

### **Code to Read Multiple Excel Files**

```java
import org.apache.poi.ss.usermodel.*;
import java.io.*;
import java.nio.file.*;
import java.util.*;

public class ReadMultipleExcelFiles {

    public static void main(String[] args) {
        // Directory containing Excel files
        String excelDirectoryPath = "excel_files"; // Replace with the path to your Excel files directory

        try {
            // Get a list of all .xlsx files in the directory
            List<File> excelFiles = getExcelFiles(excelDirectoryPath);

            if (excelFiles.isEmpty()) {
                System.out.println("No Excel files found in the directory.");
                return;
            }

            // Read and process each Excel file
            for (File excelFile : excelFiles) {
                System.out.println("Processing file: " + excelFile.getName());
                processExcelFile(excelFile);
                System.out.println("----------------------------------");
            }
        } catch (IOException e) {
            System.err.println("Error reading Excel files: " + e.getMessage());
        }
    }

    // Method to get all .xlsx files from a directory
    private static List<File> getExcelFiles(String directoryPath) throws IOException {
        List<File> excelFiles = new ArrayList<>();
        Files.walk(Paths.get(directoryPath))
                .filter(Files::isRegularFile)
                .filter(path -> path.toString().endsWith(".xlsx"))
                .forEach(path -> excelFiles.add(path.toFile()));
        return excelFiles;
    }

    // Method to read and process an Excel file
    private static void processExcelFile(File excelFile) {
        try (FileInputStream fis = new FileInputStream(excelFile);
             Workbook workbook = WorkbookFactory.create(fis)) {

            // Iterate through each sheet in the workbook
            for (Sheet sheet : workbook) {
                System.out.println("Reading sheet: " + sheet.getSheetName());

                // Iterate through each row
                for (Row row : sheet) {
                    // Read and print all cell values in the row
                    for (Cell cell : row) {
                        System.out.print(getCellValue(cell) + "\t");
                    }
                    System.out.println();
                }
            }
        } catch (Exception e) {
            System.err.println("Error processing file " + excelFile.getName() + ": " + e.getMessage());
        }
    }

    // Utility method to get cell value as a String
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
1. **Directory Scan**: 
   - The program scans the specified directory (`excelDirectoryPath`) for `.xlsx` files using `Files.walk`.
2. **Read Excel Files**:
   - For each Excel file, the program uses Apache POI to open the file and read its contents.
3. **Iterate Through Sheets and Rows**:
   - The code iterates through each sheet, row, and cell, printing the cell values to the console.
4. **Utility Method**:
   - The `getCellValue` method ensures proper handling of different cell types (e.g., strings, numbers, dates).

---

### **Output**
For each Excel file, the program:
- Prints the name of the file.
- Prints the name of each sheet.
- Displays all rows and cells in a tab-separated format.

---

### **Directory Structure Example**
```
project_directory/
│
├── excel_files/       # Directory containing Excel files
│   ├── file1.xlsx
│   ├── file2.xlsx
│   ├── file3.xlsx
│
├── src/               # Java source code
│   └── ReadMultipleExcelFiles.java
```

---

### **How to Add Operations**
To perform operations on the data:
- Modify the `processExcelFile` method.
- Add logic to filter, aggregate, or manipulate data as needed.
- You can store the data in a collection (e.g., `List`, `Map`) for further processing.

---

### **Dependencies**
Make sure you have the **Apache POI** dependencies in your `pom.xml`:

```xml
<dependencies>
    <!-- Apache POI -->
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi</artifactId>
        <version>5.2.3</version>
    </dependency>
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version>5.2.3</version>
    </dependency>
</dependencies>
```

---

Let me know what specific operations you’d like to perform on the Excel data, and I can help you implement them! 😊
