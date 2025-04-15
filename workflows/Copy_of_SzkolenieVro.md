# Copy of SzkolenieVro - Workflow Documentation

<details>
<summary><h2>Workflow Details</h2></summary>

- **Workflow Name:** Copy of SzkolenieVro
- **Workflow ID:** `19ee95f1-a9d9-4ada-8ba7-3e0384d591f2`
- **Description:** Szkolenie vRO
</details>

<details>
<summary><h2>Workflow Variables</h2></summary>

| Name | Type |
|------|------|
| fullText | string |
| var_0 | string |
| var_1 | string |
| var_2 | Properties |
</details>

<details>
<summary><h2>Workflow Inputs</h2></summary>

| Name | Type |
|------|------|
| name | string |
</details>

<details>
<summary><h2>Workflow Form</h2></summary>

| ID | Label | Data Type | Constraints | Default | Value List | Signpost |
|----|-------|-----------|-------------|---------|------------|----------|
| name | name | string | required: false; | n/a | n/a | n/a |
</details>

<details>
<summary><h2>Workflow Elements</h2></summary>

#### Element: item0
- **Display Name:** 
- **Type:** end
- **Description:** _No description provided_
---

#### Element: item1
- **Display Name:** Przywitanie
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| name | string | name |

**Output Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| fullText | string | fullText |

**Script:**

```javascript
System.log("Tresc z elementu przywitanie")
System.log("Hello " + name);

fullText = "Helo " + name;


```
---

#### Element: item2
- **Display Name:** pobranie wartosci
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| fullText | string | fullText |

**Script:**

```javascript
System.log("Tresc z elementu pobranie wartosci")
System.log(fullText)
```
---

#### Element: item3
- **Display Name:** Python script
- **Type:** task
- **Description:** Simple task with custom script capability.
- **Runtime Environment:** python:3.7
**Output Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| var_2 | Properties | var_2 |

**Script:**

```javascript
import json

def handler(context, inputs):
    jsonOut=json.dumps(inputs, separators=(',', ':'))
    print("Inputs were {0}".format(jsonOut))

    var_2 = {
      "test": "done"
    }

    return var_2

```
---

</details>

<details>
<summary><h2>Error Handlers</h2></summary>

_No error handlers defined._
</details>

<details>
<summary><h2>Linked Workflows</h2></summary>

_No linked workflows defined._
</details>

<details>
<summary><h2>Linked Actions Documentation</h2></summary>

# Actions Documentation


</details>

