Here’s a Java Maven project code to convert an Excel file to Parquet format. This implementation uses the **Apache POI** library to read Excel files and **Apache Parquet** library to write Parquet files.

### Maven Dependencies

Add the following dependencies to your `pom.xml`:

```xml
<dependencies>
    <!-- Apache POI for reading Excel files -->
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
    
    <!-- Parquet Writer -->
    <dependency>
        <groupId>org.apache.parquet</groupId>
        <artifactId>parquet-avro</artifactId>
        <version>1.13.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>3.3.6</version>
    </dependency>
</dependencies>
```

### Java Code

```java
import org.apache.parquet.hadoop.ParquetWriter;
import org.apache.parquet.hadoop.metadata.CompressionCodecName;
import org.apache.parquet.schema.MessageType;
import org.apache.parquet.schema.MessageTypeParser;
import org.apache.parquet.schema.PrimitiveType.PrimitiveTypeName;
import org.apache.parquet.schema.Types;
import org.apache.parquet.example.data.Group;
import org.apache.parquet.example.data.simple.SimpleGroup;
import org.apache.parquet.hadoop.example.GroupWriteSupport;
import org.apache.poi.ss.usermodel.*;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;

public class ExcelToParquet {
    public static void main(String[] args) {
        String excelFilePath = "input.xlsx";
        String parquetFilePath = "output.parquet";

        try (FileInputStream fileInputStream = new FileInputStream(new File(excelFilePath))) {
            Workbook workbook = WorkbookFactory.create(fileInputStream);
            Sheet sheet = workbook.getSheetAt(0); // Read the first sheet
            
            MessageType schema = generateParquetSchema(sheet);
            writeParquet(sheet, schema, parquetFilePath);
            
            System.out.println("Conversion completed: " + parquetFilePath);
        } catch (IOException e) {
            System.err.println("Error processing the files: " + e.getMessage());
            e.printStackTrace();
        }
    }

    private static MessageType generateParquetSchema(Sheet sheet) {
        Row headerRow = sheet.getRow(0); // Assume the first row as header
        Types.MessageTypeBuilder builder = Types.buildMessage();
        for (Cell cell : headerRow) {
            String columnName = cell.getStringCellValue();
            builder = builder.optional(PrimitiveTypeName.BINARY).named(columnName);
        }
        return builder.named("ExcelSchema");
    }

    private static void writeParquet(Sheet sheet, MessageType schema, String parquetFilePath) throws IOException {
        GroupWriteSupport.setSchema(schema, new org.apache.hadoop.conf.Configuration());
        try (ParquetWriter<Group> writer = ExampleParquetWriter.builder(new org.apache.hadoop.fs.Path(parquetFilePath))
                .withType(schema)
                .withCompressionCodec(CompressionCodecName.SNAPPY)
                .withWriteMode(org.apache.parquet.hadoop.ParquetFileWriter.Mode.OVERWRITE)
                .build()) {

            for (int i = 1; i <= sheet.getLastRowNum(); i++) { // Start from the second row
                Row row = sheet.getRow(i);
                Group group = new SimpleGroup(schema);
                for (int j = 0; j < row.getLastCellNum(); j++) {
                    Cell cell = row.getCell(j);
                    if (cell != null) {
                        group.add(j, cell.toString());
                    } else {
                        group.add(j, ""); // Add empty string for null cells
                    }
                }
                writer.write(group);
            }
        }
    }
}
```

### Key Points:
1. **Excel Reading**: The `Apache POI` library reads Excel files, including `.xlsx` and `.xls` formats.
2. **Parquet Writing**: The `Apache Parquet` library handles schema and file generation. Here, a simple binary schema is created for all columns.
3. **Compression**: Parquet files are compressed using Snappy for efficiency.

### Steps to Execute:
1. Save your Excel file as `input.xlsx` in the project directory.
2. Run the code, and the output Parquet file `output.parquet` will be generated.

This code assumes that the Excel file's first row contains headers, and the rest contains data. Modify the schema generation logic if needed for specific data types.
