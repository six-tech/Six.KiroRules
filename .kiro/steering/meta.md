---
description: A meta-steering-rule for using Kiro to update Kiro's steering *.md files in this repository.
fileMatchPattern: "steering/**/*.md"
inclusion: always
---

# Kiro Steering Rules

This document outlines the meta-steering-rules for Kiro operations in this repository.

## Line Ending & Formatting Rules

1. **Line Endings**
   - Don't modify existing line endings
   - Preserve CRLF/LF as found in the original file

2. **Whitespace**
   - Preserve trailing whitespace where it exists
   - Maintain existing indentation style (spaces vs tabs)
   - Keep line length consistent with existing code

## File Handling

1. **Steering MD Files**
   - All steering files are located in `steering/` directory
   - Always commit changes to steering `*.md` files directly
   - Ensure changes to steering `*.md` files are synchronized with their implementation
   - Maintain consistent formatting within rules in steering `.md` files
   - Prefer modifying existing files over creating new ones unless explicitly requested
   - When adding new content, first check if it fits within an existing steering file
   - When referencing other steering `.md` file or any other file in the repository from this steering file, use Kiro's file reference format: #[[file:{path-to-the-file}]]. For example: #[[file:api/openapi.yaml]] or #[[file:components/button.tsx]]. This ensures that Kiro always uses the latest project details when generating suggestions or guidance.   


2. **File Creation**

   - Only create new steering files when:
     - Explicitly requested by the user
     - The content doesn't logically fit in any existing steering file
     - Creating a new file would significantly improve organization and clarity
   - Document the rationale for creating new files in their README.md

## Code Style & Structure

1. **Formatting**
   - Preserve existing code formatting patterns
   - Maintain consistent naming conventions
   - Keep indentation consistent with the file style

2. **Organization**
   - Keep the file structure consistent
   - Don't move files without updating all references
   - Maintain the existing import /include order
  
3. **Frontmatter**
   - Always add `description` with short but informative description of the steering rule
   - Always add `fileMatchPattern` with glob patters in "". Example: `fileMatchPattern: "*.sln", "*.slnx", "global.json", "./Directory.Build.props"`
   - Always add `inclusion` set as `manual` if no or empty `fileMatchPattern` is provided. Else, use `fileMatch`. Note, available values are: `manual`, `fileMatch` or `always`

## Rule Content

1. **Examples**
    - Always include "DO" and "DONT'T" examples when showing specific changes to files or syntax
    - Include helpful comments that explain why an example is a "DO" or a "DONT'T"    
    - Group steering rules in logical groups using heading elements


# End of Kiro Steering File