*
### getmodifiedFields() Method

```java
CplanUpdateData getModifiedFields(T current, T existing) {
    Field[] declaredFields = current.getClass().getDeclaredFields();
    final List<CplanUpdateMetaData> cplanComparisonMetaDataList = new ArrayList<>();

    for (Field field : declaredFields) {
        field.setAccessible(true);
        if (!IGNORED_FIELDS.contains(field.getName())) {
            try {
                Object currentValue = field.get(current);
                Object existingValue = field.get(existing);
                if (!Objects.equals(currentValue, existingValue)) {
                    cplanComparisonMetaDataList.add(
                        new CplanUpdateMetaData(field.getName(), existingValue, currentValue)
                    );
                }
            } catch (Exception e) {
                log.warn("Unable to compare field {}", field.getName(), e);
            }
        }
    }

    return CplanUpdateData.builder()
        .key(current.getKey()) // assuming T has getKey()
        .cplanUpdateMetaDataList(cplanComparisonMetaDataList)
        .build();
}



## Feature for this method:

Absolutely! Here's a **numbered list** of all the key features of the `getModifiedFields` method along with brief explanations for each:

---

### 🔢 **Features of `getModifiedFields` Method (with Explanations):**

1. **Uses Java Reflection to Access Fields**  
   → The method uses `current.getClass().getDeclaredFields()` to get all fields of the object at runtime, regardless of their access modifier (private, public, etc.). This allows dynamic comparison of any object's fields.

2. **Compares Two Objects Field-by-Field**  
   → It compares each field in the `current` object with the corresponding field in the `existing` object to detect any differences.

3. **Skips Fields Marked as Ignored**  
   → If a field's name is present in the `IGNORED_FIELDS` list, the method skips comparing it. This is useful for ignoring metadata or audit fields like `lastUpdatedBy`.

4. **Handles Private Fields Using `setAccessible(true)`**  
   → This line allows access to private fields using reflection. Without it, Java would throw an `IllegalAccessException`.

5. **Performs Null-Safe Equality Check**  
   → It uses `Objects.equals(a, b)` which safely handles null values during comparison (i.e., it returns true if both are null).

6. **Builds a List of Modified Fields**  
   → When a difference is found, a new `CplanUpdateMetaData` object is created and added to a list that records all changed fields.

7. **Logs Warning on Exceptions**  
   → If any exception occurs while comparing fields (like `IllegalAccessException`), it logs a warning and continues without crashing the program.

8. **Returns a Custom Object with Comparison Summary**  
   → At the end, it builds and returns a `CplanUpdateData` object that contains:
   - The key (likely a unique identifier of the object)
   - The list of all modified fields and their before/after values

---

Let me know if you want this list converted to Markdown format, or explained more deeply!

-------------------


### Explination of the method in detail


```java

Absolutely Joseph! Let's now break down the `getModifiedFields(...)` method you saved — **line by line** and in **super simple terms**. I’ll also explain the Java concepts behind each part so you understand **what it's doing and why**. 🧠

---

### 📌 First, what is this method for?

This method is used to:
> Compare two objects (`current` and `existing`) of the same class, and find **which fields are different** between them.

It returns:
✅ A list of those changed fields (with old and new values), wrapped in a `CplanUpdateData` object.

---

## 🔍 Full Method Breakdown

### 🔹 Line 1:
```java
Field[] declaredFields = current.getClass().getDeclaredFields();
```

🧠 **What it does:**  
Gets all the declared fields of the current object (like `name`, `status`, `type`, etc.)

💡 **Java concept:**  
Reflection API — `Field` comes from `java.lang.reflect`. It lets you examine and access fields **dynamically at runtime**, even if you don't know the names at compile time.

---

### 🔹 Line 2:
```java
final List<CplanUpdateMetaData> cplanComparisonMetaDataList = new ArrayList<>();
```

🧠 **What it does:**  
Creates a list to collect all the fields that are different between the current and existing objects.

💡 **Java concept:**  
You’re using **List** to store dynamic field differences, and `CplanUpdateMetaData` likely has three things:
- Field name
- Old value
- New value

---

### 🔹 Line 4:
```java
for (Field field : declaredFields) {
```

🧠 **What it does:**  
Loop through each field (like a `for-each` on all variables of the class).

---

### 🔹 Line 5:
```java
field.setAccessible(true);
```

🧠 **What it does:**  
Makes private fields accessible via reflection.

💡 **Java concept:**  
Normally, you can’t access private fields like `private String status`, but `setAccessible(true)` lets you **bypass access checks**.

---

### 🔹 Line 6:
```java
if (!IGNORED_FIELDS.contains(field.getName())) {
```

🧠 **What it does:**  
Skips fields like `"createTimestamp"` or `"updateTimestamp"` — we usually don’t care if these change.

💡 **Java concept:**  
This is a **filtering mechanism**. `IGNORED_FIELDS` is a `List<String>` that holds names of fields you want to skip.

---

### 🔹 Lines 7–13: Compare field values

```java
try {
    Object currentValue = field.get(current);
    Object existingValue = field.get(existing);
    if (!Objects.equals(currentValue, existingValue)) {
        cplanComparisonMetaDataList.add(
            new CplanUpdateMetaData(field.getName(), existingValue, currentValue)
        );
    }
} catch (Exception e) {
    log.warn("Unable to compare field {}", field.getName(), e);
}
```

### 🔸 Line 8–9:
```java
Object currentValue = field.get(current);
Object existingValue = field.get(existing);
```

🧠 **What it does:**  
Gets the actual value of the field from both `current` and `existing`.

💡 **Java concept:**  
Again, this is using **reflection** to access field values dynamically.

---

### 🔸 Line 10:
```java
if (!Objects.equals(currentValue, existingValue)) {
```

🧠 **What it does:**  
Checks if the values are different.  
If yes → that means the field was updated!

💡 **Java concept:**  
`Objects.equals()` safely compares even if one or both values are `null`.

---

### 🔸 Line 11–12:
```java
cplanComparisonMetaDataList.add(
    new CplanUpdateMetaData(field.getName(), existingValue, currentValue)
);
```

🧠 **What it does:**  
Creates an object representing the changed field and adds it to the result list.

💡 **Java concept:**  
You’re collecting all changes in a structured way using a helper class.

---

### 🔸 Line 14–15:
```java
} catch (Exception e) {
    log.warn("Unable to compare field {}", field.getName(), e);
}
```

🧠 **What it does:**  
If something goes wrong (like reflection failing), log a warning but don’t crash.

💡 **Java concept:**  
This is standard **error handling** using `try-catch`.

---

### 🔹 Final Return Block:

```java
return CplanUpdateData.builder()
    .key(current.getKey()) // assuming T has getKey()
    .cplanUpdateMetaDataList(cplanComparisonMetaDataList)
    .build();
```

🧠 **What it does:**

- Creates a new `CplanUpdateData` object
- Fills it with:
  - The key for the current object
  - The list of fields that changed

💡 **Java concept:**
- You're using the **Builder pattern** (common in Lombok or fluent Java)
- It helps you build objects in a clean way

---

## 🎓 Real-Life Analogy

Imagine comparing two resumes:

- Old resume (DB copy)
- New resume (CPlan copy)

You check:
- Has the **job title** changed?
- Did the **skills** update?

You **write down any differences** (this is `CplanUpdateMetaData`)  
And finally, **bundle those notes** with the person’s ID (this is `CplanUpdateData`)

---

## ✅ Summary of What This Method Does:

| Step | Purpose |
|------|---------|
| Get all fields of the object | Use reflection |
| Ignore some fields like timestamps | Optional filtering |
| Compare each field’s value | Using `Objects.equals(...)` |
| Save changed ones | In `CplanUpdateMetaData` |
| Return result | As a `CplanUpdateData` object |

---

Let me know if you'd like me to:
- Show how this looks in real code with an example object
- Explain `CplanUpdateMetaData` class next
- Or simulate what gets added when values change

You're absolutely on fire with these questions, Joseph! 🔥