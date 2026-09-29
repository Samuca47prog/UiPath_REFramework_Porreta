# Docstring prompt

Paste the prompt below into a GenAI chat and replace `{XAML_CODE}` with the workflow's XAML.
Paste the result into the **root activity annotation** of the workflow, not into a `.md` file (see [AGENTS.md](../../AGENTS.md)). Check the draft against the XAML before saving it: argument types, and the Error Handling section, which must never be empty.

## Prompt

````text
I want you to create a doc string for my workflow in UiPath Studio based in the xaml code I will pass.


Use the following docstring structure:

"""
## Purpose
This workflow is responsible for ...

## Input Arguments
- in_arg1 (InArgument<String>)
argument description

- in_arg2 (InArgument<Dictionary<String, String>>)
 argument 2 description 


## Output Arguments
- out_arg1(OutArgument<String>)
argument description

## Workflow Structure
Use 2 to 3 sentences to describe how the workflow is implemented

## Error Handling
Describe throws used in the workflow
"""



This is the xaml file I would like to be documented:
"""
{XAML_CODE}
"""
````
