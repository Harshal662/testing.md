```
    public static String formatString(String input) {
        // Remove "IN" at the beginning if present
        if (input.startsWith("IN ")) {
            input = input.substring(3).trim();
        }

        // Replace "<>" with "not"
        input = input.replace("<>", "not");

        // Remove trailing "s" from words (by checking each word)
        String[] words = input.split(" ");
        StringBuilder formattedInput = new StringBuilder();
        
        for (String word : words) {
            // If the word ends with 's', remove the 's'
            if (word.endsWith("s")) {
                word = word.substring(0, word.length() - 1);
            }
            formattedInput.append(word).append(" ");
        }

        // Return the formatted string without trailing spaces
        return formattedInput.toString().trim();
    }
```
