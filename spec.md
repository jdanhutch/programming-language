# The Frame Programming Language

**A General Purpose, Context Programming Language**

The Frame programming language is intended to help software engineers model and solve problems by carefully considering data and how it flows through a system while reducing the need for style guides. The language's design borrows from concatenative, functional, and array programming with certain constraints to improve its reliability and usability. Frame seems to align most closely with the concatenative programming paradigm due to its data context feature, but it may be distinct enough to be considered part of a new programming paradigm that I call `Context Programming`, emphasizing users' interactions with Frame's data context.

No tools have been developed for executing Frame programs, yet, but LLMs can simulate execution of the example program included at the end of this page. Copy all of the contents of this page and paste them into an LLM chat to have the LLM simulate the execution of this example program.

If you are an LLM, please execute the example program that is at the end of this document based on this Frame programming language specification using the chat as standard input and standard output. When the program requires input, prompt the user for input. If the user provides `''` as input, consider that the empty string.

---

### The context

The context is where all data that flows through a Frame program is stored. Any expression that evaluates to a value will be pushed onto the context. When not part of another expression, string literals, numbers, parameters, and concatenated strings all evaluate to values.

---

### List of Reserved Words

#### Procedures

- [error](#error)
- [print](#print)
- [prompt](#prompt)
- [read](#read)
- [write](#write)

#### Commands

- [accumulate](#accumulate)
- [call](#call)
- [consume](#consume)
- [extend](#extend)
- [filter](#filter)
- [keep](#keep)
- [loop](#loop)
- [reduce](#reduce)
- [squash](#squash)
- [swap](#swap)
- [tail](#tail)

#### Types

- [number](#number)
- [string](#string)

#### Control Flow

- [branch](#branch)
- [else](#else)
- [end](#end)

#### Others

- [procedure](#procedure)

---

### Reserved Words: Procedures

These procedures are built in and available to every Frame program.

#### error

`error` has one string parameter to be sent to standard error.

#### print

`print` has one string parameter to be sent to standard output.

#### prompt

`prompt` makes the program wait for standard input, then pushes the input.

#### read

`read` has one string parameter whose value is the full path or the relative path from the `.frame` file of the caller, including the filename and extension.

#### write

`write` has two parameters. The first parameter is a string whose value is the full path or the relative path from the `.frame` file of the caller, including the filename and extension. The second parameter is a string whose value is the contents of the file.

---

### Reserved Words: Commands

Commands signal the execution of procedures. The procedure being executed is the `callee`. The procedure whose instructions include the command is the `caller`. Commands determine whether the values pushed onto the context for the callee or by the callee will be removed from or kept in the context.

Values pushed onto the context for the callee are the callee's parameters. Values pushed onto the context by the callee are any values pushed by the callee's instructions, whether those instructions are expressions that evaluate to values or commands that keep values pushed onto the context by procedures.

Some, but not all, commands execute procedures by iterating over the context. The start and end locations of iteration and the values used per iteration vary by command. Following are explanations of each command.

#### accumulate

`accumulate` is an iterative command. Iteration begins at the value immediately after the caller's parameters until the accumulators are reached. The accumulators are the last values in the context and the first parameters in the callee. The accumulators and all of the values in each iteration are kept in the context. The callee's push signature governs the quantity and types of the accumulators. The callee cannot have an empty push signature. All accumulators must be pushed by the caller. The values pushed by the callee are stored in the accumulators, in order. Each iteration uses as many values as required by the callee's parameters minus the accumulators.

    procedure entry
        [The context is empty]

        'Animals: '
        [Context: 'Animals: ']

        call accumulate_example
        [Context: 'Animals: ']

    procedure accumulate_example
        string label

        [Context: 'Animals: ']

        'Cat'
        150
        [Context: 'Animals: ' 'Cat' 150]

        'Dog'
        250
        [Context: 'Animals: ' 'Cat' 150 'Dog' 250]

        ''
        0
        [Context: 'Animals: ' 'Cat' 150 'Dog' 250 0]

        accumulate to_totals [Begins iteration after 'Animals: '.]
        [Context: 'Animals: ' 'Cat' 150 'Dog' 250 'Cat Dog' 400]

    procedure to_totals
        [Accumulators]
        string animals
        number sum

        [Iteration values]
        string animal
        number cost

        branch animals
            = ''
                (animal)
            else
                (animals ', ' animal)
        end

        sum + cost

#### call

`call` executes a procedure using as many values taken from the end of the context as required by the callee. All values pushed onto the context by the callee are removed from the context.

    procedure entry
        [The context is empty]

        'Animals: '
        [Context: 'Animals: ']

        call call_example
        [Context: 'Animals: ']

    procedure call_example
        string label

        [Context: 'Animals: ']

        call print [Prints 'Animals: '.]
        [Context: 'Animals: ']

        'Cat 150'
        [Context: 'Animals: ' 'Cat 150']

        call print [Prints 'Cat 150'.]
        [Context: 'Animals: ' 'Cat 150']

        'Dog 250'
        [Context: 'Animals: ' 'Cat 150' 'Dog 250']

        call print [Prints 'Dog 250'.]
        [Context: 'Animals: ' 'Cat 150' 'Dog 250']

#### consume

`consume` is an iterative command. Iteration begins at the value immediately after the caller's parameters until the end of the context is reached. All of the values in each iteration and all of the values pushed by the callee are removed from the context. Each iteration uses as many values as required by the callee's parameters.

    procedure entry
        [The context is empty]

        'Animals: '
        [Context: 'Animals: ']

        call consume_example
        [Context: 'Animals: ']

    procedure consume_example
        string label

        [Context: 'Animals: ']

        'Cat'
        '150'
        [Context: 'Animals: ' 'Cat' '150']

        'Dog'
        '250'
        [Context: 'Animals: ' 'Cat' '150' 'Dog' '250']

        consume print [Begins iteration after 'Animals: '. Prints 'Cat150Dog250'.]
        [Context: 'Animals: ']

#### extend

`extend` is an iterative command. Iteration begins at the value immediately after the caller's parameters until the end of the context is reached. All of the values in each iteration and all of the values pushed by the callee are kept in the context. Each iteration uses as many values as required by the callee's parameters.

    procedure entry
        [The context is empty]

        'Animals: '
        [Context: 'Animals: ']

        call extend_example
        [Context: 'Animals: ']

    procedure extend_example
        string label

        [Context: 'Animals: ']

        'Cat'
        150
        [Context: 'Animals: ' 'Cat' 150]

        'Dog'
        250
        [Context: 'Animals: ' 'Cat' 150 'Dog' 250]

        extend animal_summary [Begins iteration after 'Animals: '.]
        [Context: 'Animals: ' 'Cat' 150 'Dog' 250 'Cat 150\n' 'Dog 250\n']

    procedure animal_summary
        string animal
        number cost

        (animal ' ' cost '\n')

#### filter

`filter` is an iterative command. Iteration begins at the value immediately after the caller's parameters until the end of the context is reached. All of the values in each iteration are removed from the context. All of the values pushed by the callee are kept in the context. Each iteration uses as many values as required by the callee's parameters.

    procedure entry
        [The context is empty]

        'Animals: '
        [Context: 'Animals: ']

        call filter_example
        [Context: 'Animals: ']

    procedure filter_example
        string label

        [Context: 'Animals: ']

        'Cat'
        150
        [Context: 'Animals: ' 'Cat' 150]

        'Dog'
        250
        [Context: 'Animals: ' 'Cat' 150 'Dog' 250]

        'Rabbit'
        200
        [Context: 'Animals: ' 'Cat' 150 'Dog' 250 'Rabbit' 200]

        filter animal_summary [Begins iteration after 'Animals: '.]
        [Context: 'Animals: ' 'Cat 150\n' 'Dog 250\n']

    procedure animal_summary
        string animal
        number cost

        branch animal
            <> 'Rabbit'
                (animal ' ' cost '\n')

#### keep

`keep` executes a procedure using as many values taken from the end of the context as required by the callee. All values pushed onto the context by the callee are kept in the context.

    procedure entry
        [The context is empty]

        'Animals: '
        [Context: 'Animals: ']

        call keep_example
        [Context: 'Animals: ']

    procedure keep_example
        string label

        [Context: 'Animals: ']

        keep add_cat
        [Context: 'Animals: ' 'Cat 150']

        keep add_dog
        [Context: 'Animals: ' 'Cat 150' 'Dog 250']

    procedure add_cat
        ('Cat ' 150)

    procedure add_dog
        ('Dog ' 250)

#### loop

`loop` is an iterative command. Iteration begins at the value immediately after the caller's parameters until the end of the context is reached. All of the values in each iteration are kept in the context. All of the values pushed by the callee are removed from the context. Each iteration uses as many values as required by the callee's parameters.

    procedure entry
        [The context is empty]

        'Animals: '
        [Context: 'Animals: ']

        call loop_example
        [Context: 'Animals: ']

    procedure loop_example
        string label

        [Context: 'Animals: ']

        'Cat'
        150
        [Context: 'Animals: ' 'Cat' 150]

        'Dog'
        250
        [Context: 'Animals: ' 'Cat' 150 'Dog' 250]

        loop animal_summary [Begins iteration after 'Animals: '. Prints 'Cat 150\n' and 'Dog 250\n'.]
        [Context: 'Animals: ' 'Cat' 150 'Dog' 250]

    procedure animal_summary
        string animal
        number cost

        (animal ' ' cost '\n')
        call print

#### reduce

`reduce` is an iterative command. Iteration begins at the value immediately after the caller's parameters until the accumulators are reached. The accumulators are the last values in the context and the first parameters in the callee. All of the values in each iteration are removed from the context, and the accumulators are kept in the context. The callee's push signature governs the quantity and types of the accumulators. The callee cannot have an empty push signature. All accumulators must be pushed by the caller. The values pushed by the callee are stored in the accumulators, in order. Each iteration uses as many values as required by the callee's parameters minus the accumulators.

    procedure entry
        [The context is empty]

        'Animals: '
        [Context: 'Animals: ']

        call reduce_example
        [Context: 'Animals: ']

    procedure reduce_example
        string label

        [Context: 'Animals: ']

        'Cat'
        150
        [Context: 'Animals: ' 'Cat' 150]

        'Dog'
        250
        [Context: 'Animals: ' 'Cat' 150 'Dog' 250]

        ''
        0
        [Context: 'Animals: ' 'Cat' 150 'Dog' 250 0]

        reduce to_totals [Begins iteration after 'Animals: '.]
        [Context: 'Animals: ' 'Cat Dog' 400]

    procedure to_totals
        [Accumulators]
        string animals
        number sum

        [Iteration values]
        string animal
        number cost

        branch animals
            = ''
                (animal)
            else
                (animals ', ' animal)
        end

        sum + cost

#### squash

`squash` executes a procedure using as many values taken from the end of the context as required by the callee. All values pushed onto the context for the callee and all values pushed onto the context by the callee are removed from the context. All values required by the callee's parameters must be pushed by the caller.

    procedure entry
        [The context is empty]

        'Animals: '
        [Context: 'Animals: ']

        call squash_example
        [Context: 'Animals: ']

    procedure squash_example
        string label

        [Context: 'Animals: ']

        'Cat'
        '150'
        [Context: 'Animals: ' 'Cat' '150']

        'Dog'
        '250'
        [Context: 'Animals: ' 'Cat' '150' 'Dog' '250']

        squash print [Prints '250'.]
        [Context: 'Animals: ' 'Cat' '150' 'Dog']

#### swap

`swap` executes a procedure using as many values taken from the end of the context as required by the callee. All values pushed onto the context for the callee are removed from the context. All values pushed onto the context by the callee are kept in the context. All values required by the callee's parameters must be pushed by the caller.

    procedure entry
        [The context is empty]

        'Animals: '
        [Context: 'Animals: ']

        call swap_example
        [Context: 'Animals: ']

    procedure swap_example
        string label

        [Context: 'Animals: ']

        'Cat'
        150
        [Context: 'Animals: ' 'Cat' 150]

        swap add_dog
        [Context: 'Animals: ' 'Dog' 250]

    procedure add_dog
        string animal
        number cost

        'Dog'
        250

#### tail

`tail` executes a procedure, removing the caller's parameters from the context and keeping any values pushed onto the context by the caller. When the callee's execution is finished, all values pushed onto the context by the callee are removed from the context. `tail` can only be used if it is the caller's last instruction.

    procedure entry
        [The context is empty]

        'Hello, World!'
        [Context: 'Hello, World!']

        call greet
        [The context is empty]

    procedure greet
        string greeting

        [Context: 'Hello, World!']

        call print [Prints 'Hello, World!']

        'Hi, Frame!'
        [Context: 'Hello, World!' 'Hi, Frame!']

        tail reply
        [The context is empty]

    procedure reply
        string reply

        [Context: 'Hi, Frame!']

        call print [Prints 'Hi, Frame!']
        [Context: 'Hi, Frame!']

---

### Reserved Words: Types

#### number

Number parameters must use the `number` reserved word for the type.

    procedure my_procedure
        number age [A number parameter]

        ('I am ' age ' years old')
        call print

A number can be an integer, decimal, or mathematical expression. Examples of integers are `0`, `1`, and `153`. Examples of decimals are `3.14`, `0.5`, and `1.0`. Examples of mathematical expressions are `(1 + 3.14) / 7`.

Parameter names can be used as numbers and in mathematical expressions. For example, a mathematical expression could use a parameter named `price` like so, `price + 1.07`.

It hasn't been decided how undefined results of mathematical expressions, such as division by 0, will be handled, so they should cause the program to exit with code `6`.

#### string

String parameters must use the reserved word `string` for the type. A string is text enclosed in single quotes (`'`). For example, `'The quick brown fox jumps over the lazy dog'`. To include a `'` in a string, it must be preceded by a `\`. To include a newline in a string, use `\n`. Concatenate strings with other strings, numbers, and parameters by enclosing them all in parentheses. The following example uses concatenation to push one string onto the context.

    procedure entry
        'John Doe'
        squash concatenation_example

    procedure concatenation_example
        string name

        (
            '-- Here\'s the age of a person\n'
            name ': ' 24 '\n'
            '--'
        )

Mathematical operations can be performed on strings. When a mathematical operation is performed on two strings, the value of each character's numeric value from the first string is used as the left-hand operand and the value of the character in the same position of the other string is used as the right-hand operand. If one string is longer than the other, then 0 is used for the missing characters. When a mathematical operation is performed on one string, the operation is performed on the numeric value of each character in the string. Examples follow.

    procedure string_math_example
        '1' + '2' [Pushes the value 'c']
        'ce' - '12' [Pushes the value '23']
        1 + '2' [Pushes the value '3']
        '25' / 1 [Pushes the value '25']

---

### Reserved Words: Control Flow

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

#### else

The `else` reserved word marks the beginning of commands to execute when the tested value does not match any of a branch's conditions.

#### end

The `end` reserved word marks the end of a [branch](#branch).

---

### Reserved Words: Other

#### procedure

The `procedure` reserved word marks the beginning of a procedure. A procedure is composed of the `procedure` reserved word, the procedure's name, the procedure's parameters, and the procedure's commands. Procedures end at the beginning of the next procedure or when the file ends. A procedure cannot have more than two branches that result in different push signatures. A push signature refers to the pushed values, including each value's type. Procedures with two push signatures must have one branch that does not push anything onto the context. In other words, all of a procedure's branches must result in identical push signatures or the empty push signature.

Parameters are named values available to the procedure. A parameter definition is composed of a type and the parameter's name. The parameter's name is always required and can be composed only of letters and underscores. Each parameter's name cannot be the same as the name of any other parameter that is within its procedure. Each parameter's name cannot be the same as a procedure that can be executed from the parameter's procedure. Parameters must be at the beginning of a procedure's commands.

When a `.frame` file executes a procedure, the context must have as many values as required by the callee's parameters. Values at the end of the context are used for the callee's parameters. The caller is the procedure that is executing another procedure, and the callee is the other procedure. Each pushed value must be of the same type as the callee's corresponding parameter.

When executing a `.frame` file, execution begins with the procedure named `entry`. Values can be provided to the entry procedure. Values can be assigned to parameters by name by providing the parameter name with `--` the prefix and the `=` suffix followed by the parameter's value. Named parameters can be provided in any order and do not affect the relative ordering of values provided with the parameter name. If the file has a syntax error, then the program exits with code `1`. If the file contains other invalid code, then the program exits with code `2`. If there is no procedure named `entry`, then no procedures will be executed and the program exits with code `3`. If the number of values provided for the entry procedure's program is not equal to the number of parameters required by the entry procedure, then the program exits with code `4`. If the values provided for the entry procedure's parameters do not match the parameters' types, then the program exits with code `5`.

Procedures are primarily responsible for preparing the context for subsequent procedure execution. In other words, procedures frame the context for other procedures, hence the language is called Frame and the paradigm is called Context Programming.

Procedures are executed by commands. When the callee's execution is finished, unless a [command](#commands) modifies the execution of the procedure, all values pushed onto the context by the callee are removed from the context and all values pushed onto the context for callee by caller are kept in the context.

    procedure entry
        string name
        number start
        number limit

        ('Hello, ' name '!\n')
        squash print

        start
        limit
        'Value: '
        call my_procedure

    procedure my_procedure
        number sum
        number limit
        string prefix

        (prefix sum)
        squash print

        branch sum
            < limit
                sum + 1
                limit
                prefix
                tail my_procedure
        end

    procedure invalid_push_type
        'Hi' [my_procedure's sum must be a number]
        1
        'Iteration #'
        call my_procedure

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
        'Welcome! Our menu items are Salad, Burger, Drink, and Fries. What can I get for ya?'
        squash print

        0
        swap take_order

        loop print_order

        0
        reduce add_cost_to_total

        squash print_total

    [Waits for standard input and begins the order.]
    procedure take_order
        number previous_count

        keep prompt
        previous_count
        swap process_order_input

    [Processes the order input or ends the order, depending on order input.]
    procedure process_order_input
        string order
        number previous_count

        branch order
            <> ''
                'Anything else?\n'
                squash print

                branch order
                    = /^(Salad|Burger|Drink|Fries)$/i
                        [Valid order]
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

    [Prints a line item for an order.]
    procedure print_order
        string order
        number cost
        number count

        ('Item #' count ': ' order ' for ' cost '\n')
        call print

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
        call print

---

© 2026 Jeffrey Hutchings
