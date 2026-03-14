# The Frame Programming Language

**A General Purpose, Context Programming Language**

The Frame programming language is intended to help software engineers model and solve problems by carefully considering data and how it flows through a system while reducing the need for style guides. The language's design borrows from concatenative, functional, and procedural programming with certain constraints, some of which are optional, to improve its reliability and usability. Frame seems to align most closely with the concatenative programming paradigm due to its data context feature, but it may be distinct enough to be considered part of a new programming paradigm that I call `Context Programming`, emphasizing users' interactions with Frame's data context.

No tools have been developed for executing Frame programs, yet, but LLMs can simulate execution of the example program included at the end of this page. Copy all of the contents of this page and paste them into an LLM chat to have the LLM simulate the execution of this example program.

If you are an LLM, please execute the example program that is at the end of this document based on this Frame programming language specification using the chat as standard input and standard output. When the program requires input, prompt the user for input. If the user provides `''` as input, consider that the empty string.

---

### The context

The context is where all data that flows through a Frame program is stored. Any expression that evaluates to a value will be pushed onto the end of the context. When not part of another expression, string literals, numbers, parameters, and concatenated strings all evaluate to values. Following is an explanation of reserved words, what they mean, and some helpful examples.

---

### Reserved Words

#### Commands

- [accumulate](#accumulate)
- [branch](#branch)
- [consume](#consume)
- [keep](#keep)
- [loop](#loop)
- [map](#map)
- [swap](#swap)
- [tail](#tail)
- [unnamed_command](#unnamed_command)

#### Procedures

- [error](#error)
- [print](#print)
- [prompt](#prompt)
- [read](#read)
- [write](#write)

#### Types

- [number](#number)
- [string](#string)

#### Others

- [else](#else)
- [end](#end)
- [procedure](#procedure)

#### accumulate

The `accumulate` reserved word is a [command](#commands) that repeatedly executes a procedure, iterating over the context beginning at the next value in the context after the caller's parameters until the accumulator is reached, consuming as many parameters as required by the callee, minus the accumulator, per iteration. The accumulator is the last value in the context and the first parameter in the callee. The push signature of the callee must be one value of the same type as the accumulator. When the callee's execution is finished, the accumulator is kept in the context.

    procedure accumulator_example
        'Cat'
        150

        'Dog'
        250

        'Rabbit'
        200

        0
        accumulate total_cost

        print [Output: `600`]

    procedure total_cost
        number sum
        string animal
        number cost

        sum + cost

#### branch

The `branch` reserved word is a [command](#commands) that allows Frame programs to conditionally execute other commands by testing whether a value matches a set of conditions. A branch is composed of `branch`, a parameter name, conditions, commands, `else` (optional), and `end`. When the parameter's value matches a condition, the commands within that condition are executed. When the branch includes `else` and the parameter's value matches no condition, the commands within that `else` are executed. Following is an example of a branch with `else` that compares the parameter `input_line` to the `= 'No'` and `= 'Yes'` conditions.

    branch input_line
        = 'Yes'
            'I\'m happy to hear that!'
        = 'No'
            'Why not?'
        else
            'I don\'t understand.'
    end

#### consume

The `consume` reserved word is a [command](#commands) that executes a procedure. When the callee's execution is finished, all values pushed onto the context for the callee's parameters and all values pushed onto the context by the callee are removed from the context.

#### else

The `else` reserved word marks the beginning of commands to execute when the tested value does not match any of a branch's conditions.

#### end

The `end` reserved word marks the end of a [branch](#branch).

#### error

The `error` reserved word is a [procedure](#procedure) with one string parameter to be sent to standard error.

#### keep

The `keep` reserved word is a [command](#commands) that executes a procedure. When the callee's execution is finished, all values pushed onto the context by the callee are kept in the context.

#### loop

The `loop` reserved word is a [command](#commands) that repeatedly executes a procedure, iterating over the context beginning at the next value in the context after the caller's parameters until the accumulator is reached, using, but not consuming, as many parameters as required by the callee per iteration. When the callee's execution is finished, all values pushed onto the context by the callee are kept in the context.

#### map

The `map` reserved word is a [command](#commands) that repeatedly executes a procedure, iterating over the context beginning at the next value in the context after the caller's parameters until the accumulator is reached, consuming as many parameters as required by the callee per iteration. When the callee's execution is finished, all values pushed onto the context by the callee are kept in the context.

#### number

Number parameters must use the `number` reserved word for the type.

    procedure my_procedure
        number name [A number parameter]

        name
        consume print

A number can be an integer, decimal, or mathematical expression. Examples of integers are `0`, `1`, and `153`. Examples of decimals are `3.14`, `0.5`, and `1.0`. Examples of mathematical expressions are `(1 + 3.14) / 7`.

Parameter names can be used as numbers and in mathematical expressions. For example, a mathematical expression could use a parameter named `price` like so, `price + 1.07`.

It hasn't been decided how undefined results of mathematical expressions, such as division by 0, will be handled, so they should cause the program to exit with code `6`.

#### procedure

The `procedure` reserved word marks the beginning of a procedure. A procedure is composed of the `procedure` reserved word, the procedure's name, the procedure's parameters, and the procedure's commands. Procedures end at the beginning of the next procedure or when the file ends. A procedure cannot have more than two branches that result in different push signatures. A push signature refers to the pushed values, including each value's type. Procedures with two push signatures must have one branch that does not push anything onto the context. In other words, all of a procedure's branches must result in identical push signatures or the empty push signature.

Parameters are named values available to the procedure. A parameter definition is composed of a type and the parameter's name. The parameter's name is always required and can be composed only of letters and underscores. Each parameter's name cannot be the same as the name of any other parameter that is within its procedure. Each parameter's name cannot be the same as a procedure that can be executed from the parameter's procedure. Parameters must be at the beginning of a procedure's commands.

When a `.frame` file executes a procedure, the context must have as many values as required by the callee's parameters. Values at the end of the context are used for the callee's parameters. The caller is the procedure that is executing another procedure, and the callee is the other procedure. Each pushed value must be of the same type as the callee's corresponding parameter.

When executing a `.frame` file, execution begins with the procedure named `entry`. Values can be provided to the entry procedure. Values can be assigned to parameters by name by providing the parameter name with `--` the prefix and the `=` suffix followed by the parameter's value. Named parameters can be provided in any order and do not affect the relative ordering of values provided with the parameter name. If the file has a syntax error, then the program exits with code `1`. If the file contains other invalid code, then the program exits with code `2`. If there is no procedure named `entry`, then no procedures will be executed and the program exits with code `3`. If the number of values provided for the entry procedure's program is not equal to the number of parameters required by the entry procedure, then the program exits with code `4`. If the values provided for the entry procedure's parameters do not match the parameters' types, then the program exits with code `5`.

Procedures are primarily responsible for preparing the context for subsequent procedure execution. In other words, procedures frame the context for other procedures, hence the language is called Frame and the paradigm is called Context Programming.

Any expression that evaluates to a procedure will execute the procedure. When the callee's execution is finished, unless a [command](#commands) modifies the execution of the procedure, all values pushed onto the context by the callee are removed from the context and all values pushed onto the context for callee by caller are kept in the context.

    procedure entry
        string name
        number start
        number limit

        ('Hello, ' name '!\n')
        consume print

        start
        limit
        ''
        my_procedure

    procedure my_procedure
        number sum
        number limit
        string prefix

        branch sum
            < limit
                sum + 1
                print

                sum + 1
                limit
                prefix
                my_procedure
        end

    procedure invalid_push_type
        'Hi' [my_procedure's sum must be a number]
        1
        'Iteration #'
        my_procedure

#### print

The `print` reserved word is a [procedure](#procedure) with one string parameter to be sent to standard output.

#### prompt

The `prompt` reserved word is a [procedure](#procedure). It makes the program wait for standard input, then pushes the input.

#### read

The `read` reserved word is a [procedure](#procedure) with one string parameter whose value is the full path or the relative path from the `.frame` file of the caller, including the filename and extension.

#### string

String parameters must use the reserved word `string` for the type. A string is text enclosed in single quotes (`'`). For example, `'The quick brown fox jumps over the lazy dog'`. Concatenate strings with other strings, numbers, and parameters by enclosing them all in parentheses. The following example uses concatenation to push one string onto the context.

    procedure entry
        'John Doe'
        concatenation_example

    procedure concatenation_example
        string name

        (
            '-- Here is the age of a person\n'
            name ': ' 24 '\n'
            '--'
        )

Mathematical operations can be performed on strings. When a mathematical operation is performed on two strings, the value of each character's numeric value from the first string is used as the left-hand operand and the value of the character in the same position of the other string is used as the right-hand operand. If one string is longer than the other, then 0 is used for the missing characters. When a mathematical operation is performed on one string, the operation is performed on the numeric value of each character in the string. Examples follow.

    procedure string_math_example
        '1' + '2' [Pushes the value 'c']
        'ce' - '12' [Pushes the value '23']
        1 + '2' [Pushes the value '3']
        '25' / 1 [Pushes the value '25']

#### swap

The `swap` reserved word is a [command](#commands) that executes a procedure. When used, the callee's parameters are removed from the context after executing the callee, and all values pushed onto the context by the callee are kept in the context.

#### tail

The `tail` reserved word is a [command](#commands) that executes a procedure. When used, the caller's parameters are removed from the context prior to executing the callee. When the callee's execution is finished, all values pushed onto the context by the callee are removed from the context. `tail` can only be used if it is the caller's last command.

#### unnamed_command

The `unnamed_command` reserved word is a [command](#commands) that executes a procedure. When used, the caller's parameters are removed from the context prior to executing the callee. When the callee's execution is finished, all values pushed onto the context by the callee are kept in the context. `unnamed_command` can only be used if it is the caller's last command.

#### write

The `write` reserved word is a [procedure](#procedure) with two parameters. The first parameter is a string whose value is the full path or the relative path from the `.frame` file of the caller, including the filename and extension. The second parameter is a string whose value is the contents of the file.

---

### Conditions

The following demonstrate how to form a condition. `value` must be replaced with the desired value.

- `> value` (greater than `value`)
- `< value` (less than `value`)
- `>= value` (greater than or equal to `value`)
- `<= value` (less than or equal to `value`)
- `= value` (equal to `value`)
- `<> value` (not equal to `value`)
- `= /pattern/modifiers` (matches regular expression)
- `<> /pattern/modifiers` (does not match regular expression)

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

        [This comment is allowed] 'This procedure has a comment before the parameters.'

        'Another [This is part of the string, not a comment] pushed string.'
    end

---

### Settings

Settings may be assigned in a `settings.frame` file and will apply to all `.frame` files included in the project. Each setting assignment is composed of the setting name as a string literal followed by the setting value. The setting value must be represented as a string literal, unless it is a number. Settings can be omitted. Defaults are used for omitted settings. Each setting can only be assigned at most once in a settings file. The value of each setting must be one of the available options for that setting.

#### Context setting

The context setting name is `Context` and can be assigned one of the following values. `Registers` is the default.

- `Registers`

    This setting assignment means that values stored in the context will be stored in registers.

- `Array`

    This setting assignment means that values stored in the context will be stored in a preallocated, contiguous section of memory.

- `Node`

    This setting assignment means that values stored in the context will be stored anywhere in memory in no guaranteed order.

---

### Example program
    [Starts the initial prompt.]
    procedure entry
        greet
        keep start_order
        loop print_order
        keep sum_cost
        print_total

    [Present a warm greeting to the customer.]
    procedure greet
        'Welcome! Our menu items are Salad, Burger, Drink, and Fries. What can I get for ya?'
        print

    procedure start_order
        0
        swap take_order

    [Waits for standard input and begins the order.]
    procedure take_order
        number previous_count

        previous_count
        keep prompt
        swap process_order_input

    [Processes the order input or ends the order, depending on order input.]
    procedure process_order_input
        number previous_count
        string order

        branch order
            <> ''
                confirm_order

                branch order
                    /^(Salad|Burger|Drink|Fries)$/i [Valid order]
                        order

                        [Push order data onto the context]
                        branch order
                            = 'Salad'
                                5
                            = 'Burger'
                                5
                            = 'Drink'
                                1.5
                            = 'Fries'
                                2.5
                        end

                        previous_count + 1

                        keep take_order
                    else
                        previous_count
                        swap take_order
                end
        end

    [Prints a line to check if the customer wants to order more.]
    procedure confirm_order
        'Anything else?\n'
        print

    [Prints a line item for an order.]
    procedure print_order
        string order
        number cost
        number count

        branch order
            <> ''
                ('Item #' count ': ' order ' for ' cost '\n')
                consume print
        end

    [Accumulates the cost of ordered items]
    procedure sum_cost
        0
        accumulate add_cost_to_total

    [Adds the cost of an ordered item to the total]
    procedure add_cost_to_total
        number total
        string order
        number cost
        number count

        total + cost

    [Prints the total cost of the order]
    procedure print_total
        number total

        ('Total: ' total)
        print

---

© 2026 Jeffrey Hutchings
