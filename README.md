if (nahRuleValue.contains("<>")) {
        // Split the "not" values by commas or spaces
        Set<String> excludedValues = new HashSet<>();
        for (String value : nahRuleValue.split("[,\\s]+")) {
            if (value.startsWith("<>")) {
                excludedValues.add(value.substring(2).trim()); // Remove the "<>" prefix
            }
        }

        // Return true if the outputValue is not in the excluded set
        return !excludedValues.contains(outputValue.trim());
    }
