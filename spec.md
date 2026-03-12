# The Frame Programming Language

**A General Purpose, Context Programming Language**

The Frame programming language is intended to help software engineers model and solve problems by carefully considering data and how it flows through a system while reducing the need for style guides. The language's design borrows from concatenative, functional, and procedural programming with certain constraints, some of which are optional, to improve its reliability and usability. Frame seems to align most closely with the concatenative programming language due to its data context feature, but it may be distinct enough to be considered part of a new programming paradigm that I call `Context Programming`, emphasizing users' interactions with Frame's data context.

No tools have been developed for executing Frame programs, yet, but LLMs can simulate execution of the example programs included in this repository. Copy the contents of this file and an example program, paste them into an LLM chat, and ask the LLM to simulate the execution of the example program.

---

### Reserved Words

#### branch

The `branch` reserved word is a [command](#commands) that allows Frame programs to conditionally execute other commands by testing whether a value matches a set of conditions. A branch is composed of `branch`, `on`, and a parameter name, each separated with at least one character of whitespace, followed by conditions, commands, and, optionally, the `else` reserved word, and, finally, the `end` reserved word. When the parameter's value matches a condition, the commands within that condition are executed. When the branch includes `else` and the parameter's value matches no condition, the commands within that `else` are executed. Following is an example of a branch with `else` that compares the parameter `input_line` to the `= 'No'` and `= 'Yes'` conditions.

    branch on input_line
        = 'Yes'
            push 'I\'m happy to hear that!'
        = 'No'
            push 'Why not?'
        else
            push 'I don\'t understand.'
    end

#### call

The `call` reserved word is a [command](#commands) that executes a procedure. When the callee's execution is finished, all values pushed onto the context by the callee are removed from the context.

#### else

The `else` reserved word is part of [branch](#branch) and [match](#match).

When used with `branch`, `else` marks the beginning of commands to execute when the tested value does not match any of a branch's conditions. At least one character of whitespace must precede the else's first command. When `else` is immediately preceded by a parameter name, at least one character of whitespace must precede `else`.

When used with `match`, `else` immediately precedes the mapped value of the tested value.

#### end

The `end` reserved word marks the end of a [branch](#branch).

#### error

The `error` reserved word is a [procedure](#procedure) with one string parameter to be sent to standard error.

#### keep

The `keep` reserved word is a [command](#commands) that executes a procedure. When the callee's execution is finished, all values pushed onto the context by the callee are kept in the context.

#### loop

The `loop` reserved word is a [command](#commands) that repeatedly executes a procedure, iterating over the context beginning after the callee's parameters until the end of the context is reached, consuming as many parameters as required by the callee per iteration. When the callee's execution is finished, all values pushed onto the context by the callee are kept in the context.

#### match

The `match` reserved word allows Frame programs to map a one value to another value by testing whether a value matches a set of conditions. A match is composed of `branch`, `on`, and a parameter name, each separated with at least one character of whitespace, followed by conditions, values, and, optionally, the `else` reserved word. When the parameter's value matches a condition, then it is mapped to the value immediately preceded by that condition. When the match includes `else` and the parameter's value matches no condition, then it is mapped to the value immediately preceded by `else`. Following is an example pushing a value mapped by a match with `else` that compares the parameter `input_line` to the `= 'No'` and `= 'Yes'` conditions.

    push match on input_line
        = 'Yes'
            'I\'m happy to hear that!'
        = 'No'
            'Why not?'
        else
            'I don\'t understand.'

#### number

Number parameters must use the `number` reserved word for the type.

    procedure my_procedure
        number name [A number parameter]

        push name
        print

A number can be an integer, decimal, or mathematical expression. Examples of integers are `0`, `1`, and `153`. Examples of decimals are `3.14`, `0.5`, and `1.0`. Examples of mathematical expressions are `(1 + 3.14) / 7`. Any amount of whitespace can be placed between operators and operands.

Parameter names can be used as numbers and in mathematical expressions. For example, a mathematical expression could use a parameter named `price` like so, `price + 1.07`.

It hasn't been decided how undefined results of mathematical expressions, such as division by 0, will be handled, so they should cause the program to exit with code `6`.

#### on

The `on` reserved word is part of [branch](#branch) and [match](#match).

#### procedure

The `procedure` reserved word marks the beginning of a procedure. A procedure is composed of four things, each separated by at least one character of whitespace: the `procedure` reserved word, the procedure's name, the procedure's parameters, and the procedure's commands. Parameters are optional. Each procedure must have at least one command. Procedures end at the beginning of the next procedure or when the file ends.

Parameters are named values available to the procedure. A parameter definition is composed of a type, the parameter's name, and, if the parameter is optional, a type-matched default value. The parameter's name is always required and can be composed only of letters and underscores. Each parameter's name cannot be the same as the name of any other parameter that is within its procedure. Each parameter's name cannot be the same as a procedure that can be executed from the parameter's procedure. Parameters must be at the beginning of a procedure commands.

When a `.frame` file executes a procedure, the caller must push as many values onto the context as required by the callee's parameters. The caller is the procedure that is executing another procedure, and the callee is the other procedure. Each pushed value must be of the same type as the callee's corresponding parameter.

When executing a `.frame` file, execution begins with the procedure named `entry`. Values can be provided to the entry procedure. Values can be assigned to parameters by name by providing the parameter name with `--` the prefix and the `=` suffix followed by the parameter's value. Named parameters can be provided in any order and do not affect the relative ordering of values provided with the parameter name. If the file has a syntax error, then the program exits with code `1`. If the file contains other invalid code, then the program exits with code `2`. If there is no procedure named `entry`, then no procedures will be executed and the program exits with code `3`. If the number of values provided for the entry procedure's program is not equal to the number of parameters required by the entry procedure, then the program exits with code `4`. If the values provided for the entry procedure's parameters do not match the parameters' types, then the program exits with code `5`.

Procedures are primarily responsible for preparing the context for subsequent procedure execution. In other words, procedures frame the context for other procedures, hence the language is called Frame and the paradigm is called Context Programming.

    procedure entry
        string name = 'Unnamed'
        number start = 0
        number limit

        push 'Hello, 'name'!\n'
        print

        push start
        push limit
        push ''
        my_procedure

    procedure my_procedure
        number sum
        number limit
        string prefix

        branch on sum
            < limit
                push 
                print

                push sum + 1
                push limit
                push prefix
                call my_procedure
        end

    procedure invalid_push_type
        push 'Hi' [my_procedure's sum must be a number]
        push 1
        push 'Iteration #'
        my_procedure

#### print

The `print` reserved word is a [procedure](#procedure) with one string parameter to be sent to standard output.

#### prompt

The `prompt` reserved word is a [procedure](#procedure) with one optional string parameter to be sent to standard output. The parameter's default value is the empty string. The program then waits for standard input and pushes the input.

#### push

The `push` reserved word is a [command](#commands) that pushes a value onto the context.

    procedure my_procedure
        string value

        push value [Parameters can be pushed. Whitespace is required between `push` and the parameter.]
        push'' [Strings can be pushed. No whitespace is required between `push` and the string.]
        push match on value [Match can be pushed. Whitespace is required between `push` and `match`.]
            'example' 'other'

#### read

The `read` reserved word is a [procedure](#procedure) with one string parameter whose value is the full path or the relative path from the `.frame` file of the caller, including the filename and extension.

#### string

String parameters must use the reserved word `string` for the type.

    procedure my_procedure
        string name [A string parameter]

        push name
        print

A string is text enclosed in single quotes (`'`). For example, `'The quick brown fox jumps over the lazy dog'`. Strings can be concatenated with other strings, numbers, and parameters. Any amount of whitespace can be placed between strings. The following example pushes onto the context one string that is composed of two concatenated strings.

    push 'Here is a line\n'
        'And here is another line'

Parameter names can be used as strings. For example, `push 'Hello, ' name '!'` pushes onto the context one string that is composed of one string concatenated with the parameter `name` concatenated with another string.

Mathematical operations can be performed on strings. When a mathematical operation is performed on two strings, the value of each character's numeric value from the first string is used as the left-hand operand and the value of the character in the same position of the other string is used as the right-hand operand. If one string is longer than the other, then 0 is used for the missing characters. When a mathematical operation is performed on one string, the operation is performed on the numeric value of each character in the string. Examples follow.

    push '1' + '2' [Pushes the value 'c']
    push 'ce' - '12' [Pushes the value '23']
    push 1 + '2' [Pushes the value '3']
    push '25' / 1 [Pushes the value '25']

#### tail

The `tail` reserved word is a [command](#commands) that executes a procedure. When used, the caller's parameters are removed from the context prior to executing the callee. `tail` can only be used if it is the caller's last command. When the callee's execution is finished, all values pushed onto the context by the callee are removed from the context.

#### write

The `write` reserved word is a [procedure](#procedure) with two parameters. The first parameter is a string whose value is the full path or the relative path from the `.frame` file of the caller, including the filename and extension. The second parameter is a string whose value is the contents of the file.

---

### Conditions

The following demonstrate how to form a condition. `value` must be replaced with the desired value. Whitespace is ignored except in regular expressions.

- `> value` (greater than `value`)
- `< value` (less than `value`)
- `>= value` (greater than or equal to `value`)
- `<= value` (less than or equal to `value`)
- `= value` (equal to `value`)
- `<> value` (not equal to `value`)
- `/pattern/modifiers` (matches regular expression)
- `<> /pattern/modifiers` (mismatches regular expression)

---

#### Comments

Comments begin with `[` and end with `]`. Comments can span one or more lines. To include a `]` in a comment, it must be preceded by a `\`. To include a `\` in a comment, it must be preceded by a `\`. Comments are allowed anywhere whitespace is allowed.

    [Here is a single line comment]

    [
        Here is another comment.
        It spans multiple lines.
    ]

    [Someone put \] and \\ in this comment.]

    procedure my_procedure
        [This comment is allowed]
        string my_param

        push [This comment is allowed] 'This procedure has a comment before the parameters.'

        push 'Another [This is part of the string, not a comment] pushed string.'
    end

---

### Commands

The following reserved words are commands.

- [branch](#branch)
- [call](#call)
- [keep](#keep)
- [loop](#loop)
- [push](#push)
- [tail](#tail)

---

### Settings

Settings may be assigned in a `settings.frame` file and will apply to all `.frame` files included in the project. Each setting assignment is composed of the setting name followed at least one character of whitespace followed by the setting value. Settings can be omitted. Defaults are used for omitted settings. Each setting can only be assigned at most once in a settings file. The value of each setting must be one of the available options for that setting.

#### Context setting

The context setting name is `Context` and can be assigned one of the following values. `Registers` is the default.

- `Registers`

    This setting assignment means that values stored in the context will be stored in registers.

- `Array`

    This setting assignment means that values stored in the context will be stored in a preallocated, contiguous section of memory.

- `Node`

    This setting assignment means that values stored in the context will be stored anywhere in memory in no guaranteed order.

---

© 2026 Jeffrey Hutchings
