---
layout: page
title: Working with Command Line
parent: Getting started with GUAC
---

# About Guacone
guacone is a versatile command-line utility for managing guacamole recipes. It allows users to create recipes, add or remove ingredients, modify recipes, and export them to various formats. The tool also provides features for listing, viewing, and organizing multiple recipes. guacone aims to simplify the creation and management of guacamole recipes via a fast, intuitive CLI interface.

Users can store recipes locally, manipulate ingredient lists, and perform batch operations for recipe management.

## COMMANDS

### `Create`
Creates a new guacamole recipe and initializes it with a basic set of attributes. If no description is provided, an empty recipe is created with the given name.

**Syntax:**
```sh
guacone create "Recipe Name" [--description "Description of the recipe"]
```

**Options:**
- `--description "DESCRIPTION"`: Adds a brief description of the recipe.

**Example:**
```sh
guacone create "Classic Guacamole" --description "A traditional guacamole recipe with avocados, lime juice, and salt."
```

---

### `add-ingredient`
Adds an ingredient to a recipe along with its quantity and optional measurement unit.

**Syntax:**
```sh
guacone add-ingredient "Recipe Name" "Ingredient" QUANTITY [--unit UNIT]
```

**Options:**
- `--unit "UNIT"`: Optional measurement unit for the ingredient (e.g., cups, grams, tablespoons). Defaults to "pieces" if not provided.

**Example:**
```sh
guacone add-ingredient "Classic Guacamole" "Avocados" 3 --unit pieces
```

---

### `remove-ingredient`
Removes a specified ingredient from the recipe. The ingredient must match exactly (case-sensitive).

**Syntax:**
```sh
guacone remove-ingredient "Recipe Name" "Ingredient"
```

**Example:**
```sh
guacone remove-ingredient "Classic Guacamole" "Lime Juice"
```

---

### `list`
Lists all the recipes available in the local repository. Displays recipe names along with their creation dates. 

**Syntax:**
```sh
guacone list [--sort-by {name | date}] [--reverse]
```

**Options:**
- `--sort-by {name | date}`: Sorts the list by recipe name or creation date (default is name).
- `--reverse`: Reverses the order of the list.

**Example:**
```sh
guacone list --sort-by date --reverse
```

---

### `view`
Displays all details of a recipe, including the list of ingredients, their quantities, the recipe description, and creation date.

**Syntax:**
```sh
guacone view "Recipe Name"
```

**Example:**
```sh
guacone view "Classic Guacamole"
```

---

### `modify`
Modifies the name or description of an existing recipe.

**Syntax:**
```sh
guacone modify "Recipe Name" [--name "New Recipe Name"] [--description "New description"]
```

**Options:**
- `--name "NEW NAME"`: Renames the recipe.
- `--description "NEW DESCRIPTION"`: Updates the recipe description.

**Example:**
```sh
guacone modify "Classic Guacamole" --name "Best Guacamole" --description "Updated classic guacamole recipe with jalapeños."
```

---

### `export`
Exports a recipe to a specified file format. The default format is plain text.

**Syntax:**
```sh
guacone export "Recipe Name" --output FILE_PATH [--format {txt | json | md}]
```

**Options:**
- `--output "FILE_PATH"`: Specifies the file path where the recipe will be exported.
- `--format {txt | json | md}`: Exports the recipe in a chosen format (plain text, JSON, or markdown).

**Example:**
```sh
guacone export "Classic Guacamole" --output ~/recipes/classic_guacamole.txt --format txt
```

---

### `delete`
Deletes a recipe permanently from the local repository.

**Syntax:**
```sh
guacone delete "Recipe Name"
```

**Example:**
```sh
guacone delete "Classic Guacamole"
```

---

### `search`
Searches for recipes that match a given keyword in the name or description.

**Syntax:**
```sh
guacone search "Keyword" [--in {name | description | ingredients}]
```

**Options:**
- `--in {name | description | ingredients}`: Specifies where to search for the keyword (defaults to searching in the name and description).

**Example:**
```sh
guacone search "Spicy" --in name
```

---
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## GLOBAL OPTIONS

### `-h`, `--help`
Displays a help message listing available commands and their descriptions.

```sh
guacone --help
```

### `-v`, `--version`
Displays the current version of **guacone**.

```sh
guacone --version
```

### `-q`, `--quiet`
Runs the command without printing output to the console. Errors and warnings will still be shown.

```sh
guacone create "Quiet Recipe" --quiet
```

### `-y`, `--yes`
Automatically answers "yes" to any confirmation prompts.

```sh
guacone delete "Old Guacamole" --yes
```

---

## ENVIRONMENT VARIABLES

**GUACONE_HOME**  
Defines the default directory where recipe data and configurations are stored. If not set, the default location is `~/.guacone/`.

**Example:**
```sh
export GUACONE_HOME="/path/to/recipes"
```

---

## EXAMPLES

### Example 1: Create a new recipe
```sh
guacone create "Sweet Guacamole" --description "A sweet twist to the classic guacamole."
```

### Example 2: Add ingredients
```sh
guacone add-ingredient "Sweet Guacamole" "Avocados" 2
guacone add-ingredient "Sweet Guacamole" "Mango" 1 --unit pieces
```

### Example 3: View recipe details
```sh
guacone view "Sweet Guacamole"
```

### Example 4: Export recipe to Markdown format
```sh
guacone export "Sweet Guacamole" --output ~/recipes/sweet_guacamole.md --format md
```

---

## FILES

**~/.guacone/config**  
Configuration file for the **guacone** tool. This file stores default settings such as export format and the default recipe directory.

**~/.guacone/recipes/**  
Directory where all the recipe data files are stored.

---


## EXIT STATUS

**guacone** exits with the following status codes:

- **0**: Successful execution.
- **1**: General error, typically due to incorrect syntax or missing arguments.
- **2**: Recipe not found.
- **3**: Export or file-related error.

---