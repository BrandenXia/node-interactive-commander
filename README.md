# Interactive Commander

<div class="paragraph">

<span class="image"><a href="https://www.npmjs.com/package/interactive-commander" class="image"><img src="https://img.shields.io/npm/v/interactive-commander" alt="NPM Version" /></a></span> <span class="image"><a href="https://www.npmjs.com/package/interactive-commander" class="image"><img src="https://img.shields.io/npm/dm/interactive-commander" alt="Monthly Downloads" /></a></span> <span class="image"><a href="https://github.com/fardjad/node-interactive-commander/actions" class="image"><img src="https://img.shields.io/github/actions/workflow/status/fardjad/node-interactive-commander/test-and-release.yml?branch=main" alt="test-and-release Workflow Status" /></a></span>

</div>

Interactive Commander is an extension of the widely-used [Commander.js][1] library.
It seamlessly integrates configurable interactive prompts for missing options
in your CLI application, enhancing user experience with minimal effort.

![Video Demo](/media/demo.gif)

## Features

- Full compatibility with Commander.js (utilizes Commander.js under the hood
  and serves as a drop-in replacement for it)
- Customizable flags for interactive mode
- Automatic prompting for missing options in interactive mode
- Built-in [Inquirer.js][2] prompts for boolean, multiple-choice, and string
  options, along with the ability to create custom prompts
- [Support for plugins](/examples/plugin.ts)

## Installation

```bash
npm install --save interactive-commander
```

## Usage

```typescript
import { InteractiveCommand, InteractiveOption } from "interactive-commander";

const program = new InteractiveCommand();

program
  .command("pizza")
  // Detached options are interactive by default
  .option("-d, --drink", "drink")

  // Missing mandatory options won't throw an error in interactive mode
  .requiredOption("-o, --olive-oil", "olive oil")

  // Boolean InteractiveOptions will show a confirmation prompt by default
  .option("-c, --cheese", "cheese")
  .option("-C, --no-cheese", "no cheese")

  .addOption(
    new InteractiveOption("-n, --count <number>", "number of pizzas")
      // InteractiveOption supports all the methods of Commander.js' Option
      .argParser(Number)
      .default(1),
  )

  .addOption(
    new InteractiveOption(
      "--non-interactive-option <value>",
      "non-interactive option",
    )
      // Passing in undefined to prompt will disable interactive mode for this option
      .prompt(undefined)
      .default("default value"),
  )

  // InteractiveOptions with choices will show a select prompt by default
  .addOption(
    new InteractiveOption("-s, --size <size>", "size")
      .choices(["small", "medium", "large"])
      .default("medium")
      .makeOptionMandatory(),
  )

  .addOption(
    new InteractiveOption("-m, --name <string>", "your name")
      // You can use the prompt method to implement your own prompt logic
      .prompt(async (currentValue, option, command) => {
        // TODO: Implement your own prompt logic here
        const answer = "world";

        // Return the answer
        return answer;
      })
      .makeOptionMandatory(),
  )

  .addOption(
    new InteractiveOption("-r, --voucher-code <string>", "voucher code")
      // The prompt input gets validated by the argParser function
      .argParser((value) => {
        if (typeof value !== "string" || !/^\d{4}$/.test(value)) {
          throw new TypeError("Invalid voucher code");
        }

        return value;
      }),
  )

  .action((_options, cmd: InteractiveCommand) => {
    console.log("Options: %o", cmd.opts());
  });

await program
  // Enables interactive mode (when -i or --interactive is passed in)
  // This should almost always be called on the root command right before
  // calling parseAsync
  .interactive("-i, --interactive", "interactive mode")
  .parseAsync(process.argv);

// Try the following commands:
// command-name pizza
// command-name pizza -i
// command-name pizza -i --count 2
// command-name pizza -i --count 2 --no-cheese
// command-name pizza -i --name "John Doe"
// command-name pizza -i --name "John Doe" --non-interactive-option abc
```

More examples can be found in the [examples](/examples/) directory.

## Recipes

### Interactive options for the root command

Interactive options on main command won't be prompted for in interactive mode
if no subcommand is invoked. That is because Commander.js doesn't support
pre-parse (similar to `preSubcommand` hooks) hooks for the main command. As a
workaround, you can define a subcommand as the default command to achieve a
similar effect:

```typescript
import { InteractiveCommand } from "interactive-commander";

const program = new InteractiveCommand();

program
  .command("hello", { isDefault: true })
  .requiredOption("-n, --name <name>", "your name")
  .action((options) => {
    console.log("Hello %s!", options.name);
  });

await program.interactive().parseAsync(process.argv);

// Try the following commands:
// default-subcommand
// default-subcommand -n John
// default-subcommand -i
```

### Enable interactive mode by default

To enable interactive mode by default, you can define the interactive flags as
[negatable boolean options][3] (e.g. `--no-interactive`):

```typescript
const program = new InteractiveCommand();

program
  .command("hello")
  .option("-n, --name <name>", "your name")
  .action((options) => {
    console.log("Hello %s!", options.name);
  });

await program
  .interactive("-I, --no-interactive", "disable interactive mode")
  .parseAsync(process.argv);

// Try the following commands:
// no-interactive hello
// no-interactive hello -I
```

[1]: https://github.com/tj/commander.js
[2]: https://github.com/SBoudrias/Inquirer.js
[3]: https://github.com/tj/commander.js#other-option-types-negatable-boolean-and-booleanvalue
