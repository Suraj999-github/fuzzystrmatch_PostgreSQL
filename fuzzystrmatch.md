# Enable the fuzzystrmatch extension

 First, you need to enable the fuzzystrmatch extension in your PostgreSQL database if it is not already enabled.


```
CREATE EXTENSION fuzzystrmatch;
```

# PostgreSQL Function: ofac_checksanctionfunction

This function compares two text strings using the Daitch-Mokotoff phonetic algorithm. It returns a JSON object indicating whether the strings are a match and their similarity percentage.

## Function Definition

```sql
DROP FUNCTION public.ofac_checksanctionfunction(text, text);

CREATE OR REPLACE FUNCTION public.ofac_checksanctionfunction(firsttext text, secondtext text)
RETURNS json
LANGUAGE plpgsql
AS $function$
DECLARE
    code1 text[];
    code2 text[];
    match_count int := 0;
    total_count int;
    similarity_percentage float;
    is_match boolean;
    result json;
BEGIN
    -- Generate the Daitch-Mokotoff codes for both strings
    code1 := daitch_mokotoff(firsttext);
    code2 := daitch_mokotoff(secondtext);

    -- Get total count of possible comparisons
    total_count := GREATEST(array_length(code1, 1), array_length(code2, 1));

    -- If there's no phonetic codes generated, return 0% similarity
    IF total_count IS NULL THEN
        similarity_percentage := 0;
        is_match := false;
    ELSE
        -- Count the number of matching phonetic codes
        FOR i IN 1 .. array_length(code1, 1) LOOP
            IF code1[i] = ANY (code2) THEN
                match_count := match_count + 1;
            END IF;
        END LOOP;

        -- Calculate similarity percentage
        similarity_percentage := 100.0 * match_count / total_count;

        -- Determine if there is at least one match
        is_match := match_count > 0;
    END IF;

    -- Construct the JSON result
    result := json_build_object(
        'is_match', is_match,
        'similarity_percentage', similarity_percentage
    );

    RETURN result;
END;
$function$;
```

## Example Queries

### 1. Comparing "suraj" and "suraz"

```sql
SELECT ofac_checksanctionfunction('suraj','suraz');
```

**Result**: 
```json
{
    "is_match": false,
    "similarity_percentage": 0
}
```

### 2. Comparing "Pradeep" and "Pradip"

```sql
SELECT ofac_checksanctionfunction('Pradeep','Pradip');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 100
}
```

### 3. Comparing "Khattri" and "Chettri"

```sql
SELECT ofac_checksanctionfunction('Khattri','Chettri');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 50
}
```

### 4. Comparing "Sanjaya" and "Sanjay"

```sql
SELECT ofac_checksanctionfunction('Sanjaya','Sanjay');
```

**Result**:
```json
{
    "is_match": false,
    "similarity_percentage": 0
}
```

### 5. Comparing "Sanjaya" and "Sanzaya"

```sql
SELECT ofac_checksanctionfunction('Sanjaya','Sanzaya');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 50
}
```

### 6. Comparing "Kshetri" and "Chhettri"

```sql
SELECT ofac_checksanctionfunction('Kshetri', 'Chhettri');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 50
}
```


```markdown
# PostgreSQL Function: ofac_checksanctionfunction_v1

This function compares two text strings using both the Soundex and Daitch-Mokotoff phonetic algorithms. It returns a JSON object indicating whether the strings are a match and their similarity percentage.

## Function Definition

```sql
CREATE OR REPLACE FUNCTION public.ofac_checksanctionfunction_v1(firsttext text, secondtext text)
RETURNS json
LANGUAGE plpgsql
AS $function$
DECLARE
    code1 text;
    code2 text;
    dm_code1 text[];
    dm_code2 text[];
    match_count int := 0;
    total_count int;
    similarity_percentage float := 0.0;
    is_match boolean := false;
    result json;
BEGIN
    -- Generate the Soundex codes for both strings
    code1 := soundex(firsttext);
    code2 := soundex(secondtext);

    -- Check if the Soundex codes match
    IF code1 = code2 THEN
        similarity_percentage := 100.0;
        is_match := true;
    ELSE
        -- Generate the Daitch-Mokotoff codes for both strings
        dm_code1 := daitch_mokotoff(firsttext);
        dm_code2 := daitch_mokotoff(secondtext);

        -- Get total count of possible comparisons
        total_count := GREATEST(array_length(dm_code1, 1), array_length(dm_code2, 1));

        -- If there are valid phonetic codes generated, compare them
        IF total_count IS NOT NULL THEN
            -- Count the number of matching phonetic codes
            FOR i IN 1 .. array_length(dm_code1, 1) LOOP
                IF dm_code1[i] = ANY (dm_code2) THEN
                    match_count := match_count + 1;
                END IF;
            END LOOP;

            -- Calculate similarity percentage
            similarity_percentage := 100.0 * match_count / total_count;

            -- Determine if there is at least one match
            is_match := match_count > 0;
        END IF;
    END IF;

    -- Construct the JSON result
    result := json_build_object(
        'is_match', is_match,
        'similarity_percentage', similarity_percentage
    );

    RETURN result;
END;
$function$;
```

## Example Queries

### 1. Comparing "suraj" and "suraz"

```sql
SELECT ofac_checksanctionfunction_v1('suraj','suraz');
```

**Result**: 
```json
{
    "is_match": true,
    "similarity_percentage": 100
}
```

### 2. Comparing "Pradeep" and "Pradip"

```sql
SELECT ofac_checksanctionfunction_v1('Pradeep','Pradip');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 100
}
```

### 3. Comparing "Khattri" and "Chettri"

```sql
SELECT ofac_checksanctionfunction_v1('Khattri','Chettri');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 50
}
```

### 4. Comparing "Sanjaya" and "Sanjay"

```sql
SELECT ofac_checksanctionfunction_v1('Sanjaya','Sanjay');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 100
}
```

### 5. Comparing "Sanjaya" and "Sanzaya"

```sql
SELECT ofac_checksanctionfunction_v1('Sanjaya','Sanzaya');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 100
}
```

### 6. Comparing "Kshetri" and "Chhettri"

```sql
SELECT ofac_checksanctionfunction_v1('Kshetri', 'Chhettri');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 50
}
```

```markdown
# PostgreSQL Function: ofac_checksanctionfunction_v2

This function compares two text strings using the Levenshtein distance algorithm. It returns a JSON object indicating whether the strings are a match and their similarity percentage.

## Function Definition

```sql
CREATE OR REPLACE FUNCTION public.ofac_checksanctionfunction_v2(firsttext text, secondtext text)
RETURNS json
LANGUAGE plpgsql
AS $function$
DECLARE
    levenshtein_distance int;
    similarity_percentage float;
    is_match boolean;
    result json;
BEGIN
    -- Calculate the Levenshtein distance between the two strings
    levenshtein_distance := LEVENSHTEIN(firsttext, secondtext);

    -- Calculate similarity percentage based on the Levenshtein distance
    similarity_percentage := 100.0 * (1.0 - LEAST(LEVENSHTEIN(firsttext, secondtext)::float / GREATEST(length(firsttext), length(secondtext))::float, 1));

    -- Determine if the strings are considered a match
    is_match := similarity_percentage >= 70; -- Adjust the threshold as per your requirements

    -- Construct the JSON result
    result := json_build_object(
        'is_match', is_match,
        'similarity_percentage', similarity_percentage
    );

    RETURN result;
END;
$function$;
```

## Example Queries

### 1. Comparing "suraj" and "suraz"

```sql
SELECT ofac_checksanctionfunction_v2('suraj','suraz');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 80.0
}
```

### 2. Comparing "Pradeep" and "Pradip"

```sql
SELECT ofac_checksanctionfunction_v2('Pradeep','Pradip');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 71.42
}
```

### 3. Comparing "Khattri" and "Chettri"

```sql
SELECT ofac_checksanctionfunction_v2('Khattri','Chettri');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 71.42
}
```

### 4. Comparing "Sanjaya" and "Sanjay"

```sql
SELECT ofac_checksanctionfunction_v2('Sanjaya','Sanjay');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 85.71
}
```

### 5. Comparing "Sanjaya" and "Sanzaya"

```sql
SELECT ofac_checksanctionfunction_v2('Sanjaya','Sanzaya');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 85.71
}
```

### 6. Comparing "Kshetri" and "Chhettri"

```sql
SELECT public.ofac_checksanctionfunction_v2('Kshetri', 'Chhettri');
```

**Result**:
```json
{
    "is_match": false,
    "similarity_percentage": 62.5
}
```

# PostgreSQL Function: ofac_checksanctionfunction_v3

This function compares two text strings using both Metaphone and Daitch-Mokotoff phonetic algorithms. It returns a JSON object indicating whether the strings are a match and their similarity percentage.

## Function Definition

```sql
CREATE OR REPLACE FUNCTION public.ofac_checksanctionfunction_v3(firsttext text, secondtext text)
RETURNS json
LANGUAGE plpgsql
AS $function$
DECLARE
    code1 text;
    code2 text;
    dm_code1 text[];
    dm_code2 text[];
    match_count int := 0;
    total_count int;
    similarity_percentage float := 0.0;
    is_match boolean := false;
    result json;
BEGIN
    -- Generate the Metaphone codes for both strings
    code1 := metaphone(firsttext, 4); -- The number 4 limits the length of the generated code, adjust as necessary
    code2 := metaphone(secondtext, 4);

    -- Check if the Metaphone codes match
    IF code1 = code2 THEN
        similarity_percentage := 100.0;
        is_match := true;
    ELSE
        -- Generate the Daitch-Mokotoff codes for both strings
        dm_code1 := daitch_mokotoff(firsttext);
        dm_code2 := daitch_mokotoff(secondtext);

        -- Get the total count of possible comparisons
        total_count := GREATEST(array_length(dm_code1, 1), array_length(dm_code2, 1));

        -- If there are valid phonetic codes generated, compare them
        IF total_count IS NOT NULL THEN
            -- Count the number of matching phonetic codes
            FOR i IN 1 .. array_length(dm_code1, 1) LOOP
                IF dm_code1[i] = ANY (dm_code2) THEN
                    match_count := match_count + 1;
                END IF;
            END LOOP;

            -- Calculate similarity percentage
            similarity_percentage := 100.0 * match_count / total_count;

            -- Determine if there is at least one match
            is_match := match_count > 0;
        END IF;
    END IF;

    -- Construct the JSON result
    result := json_build_object(
        'is_match', is_match,
        'similarity_percentage', similarity_percentage
    );

    RETURN result;
END;
$function$;
```

## Example Queries

### 1. Comparing "suraj" and "suraz"

```sql
SELECT ofac_checksanctionfunction_v3('suraj','suraz');
```

**Result**: 
```json
{
    "is_match": false,
    "similarity_percentage": 0
}
```

### 2. Comparing "Pradeep" and "Pradip"

```sql
SELECT ofac_checksanctionfunction_v3('Pradeep','Pradip');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 100
}
```

### 3. Comparing "Khattri" and "Chettri"

```sql
SELECT ofac_checksanctionfunction_v3('Khattri','Chettri');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 50
}
```

### 4. Comparing "Sanjaya" and "Sanjay"

```sql
SELECT ofac_checksanctionfunction_v3('Sanjaya','Sanjay');
```

**Result**:
```json
{
    "is_match": false,
    "similarity_percentage": 0
}
```

### 5. Comparing "Sanjaya" and "Sanzaya"

```sql
SELECT ofac_checksanctionfunction_v3('Sanjaya','Sanzaya');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 85.71
}
```

### 6. Comparing "Kshetri" and "Chhettri"

```sql
SELECT ofac_checksanctionfunction_v3('Kshetri', 'Chhettri');
```

**Result**:
```json
{
    "is_match": true,
    "similarity_percentage": 50
}
```
