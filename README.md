<div align="center">
<h1>make.nu</h1>

[![Check](https://github.com/TD-Sky/make.nu/actions/workflows/check.yml/badge.svg?style=flat-square)](https://github.com/TD-Sky/make.nu/actions/workflows/check.yml)
[![Test](https://github.com/TD-Sky/make.nu/actions/workflows/test.yml/badge.svg?style=flat-square)](https://github.com/TD-Sky/make.nu/actions/workflows/test.yml)

<h4 style="font-weight: normal"><i>Organizing Tasks Using Nushell</i></h4>

</div>

> [!WARNING]
> Nushell cannot redirect stdin/stdout effectively at present (2025-07-27),
> so **make.nu** cannot run TUI in tasks normally.

## Why do I need make.nu?

- [x] Tired of learning various DSLs
- [x] Utilize Nushell commands



## Non-goal

`make.nu` **is not** a build system, although you can totally use it to build projects.



## Installation

`make.nu` consists of two crates:

- `nuke`: tool for launching `make.nu`;
- `nu_plugin_nuke`: provides functions to define tasks.

```console
$ cargo install --git https://github.com/TD-Sky/make.nu.git nuke nu_plugin_nuke
```



## Usage

1. Register Nushell plugin

   ```console
   $ plugin add ~/.cargo/bin/nu_plugin_nuke
   ```

2. Create file `make.nu` beginning with the following code:

   ```nu
   #!/usr/bin/env -S nu --stdin

   plugin use nuke

   alias task = nuke task
   ```

   And define your tasks there.

3. Run tasks at where the directory has `make.nu`:

   ```console
   $ nuke <TASK>
   ```

   `<TASK>` is your entry task, **make.nu** will run it and all the tasks it depends on.



## Example

Suppose we need to build a C project,

```
Student-Manager-c
├── .git
├── src
│   ├── main.c
│   ├── manager.c
│   ├── manager.h
│   ├── student.c
│   ├── student.h
│   ├── utils.c
│   ├── utils.h
│   ├── vec.c
│   └── vec.h
├── target
├── .gitignore
└── make.nu
```

The topological relationship between modules is:

- `main` depends on `utils`, `manager` directly;
- `manager` depends on `utils`, `student`, `vec`;
- `vec` depends on `student`.

I would write `make.nu` as follows:

```nu
#!/usr/bin/env -S nu --stdin

plugin use nuke

alias task = nuke task

mkdir target

task run --deps [build] {
    ./target/smc
}

task build --deps [
    main.o, manager.o, student.o, vec.o, utils.o
] --target target/smc {
    gcc -o target/smc target/*.o
}

task main.o --files [
    src/main.c
] --deps [
    manager.o, utils.o
] --target target/main.o {
    gcc -c src/main.c -I src -o target/main.o
}

task manager.o --files [
    src/manager.h, src/manager.c
] --deps [
    utils.o, student.o, vec.o
] --target target/manager.o {
    gcc -c src/manager.c -I src -o target/manager.o
}

task student.o --files [
    src/student.h, src/student.c
] --target target/student.o {
    gcc -c src/student.c -I src -o target/student.o
}

task vec.o --files [
    src/vec.h, src/vec.c
] --deps [
    student.o
] --target target/vec.o {
    gcc -c src/vec.c -I src -o target/vec.o
}

task utils.o --files [
    src/utils.h, src/utils.c
] --target target/utils.o {
    gcc -c src/utils.c -I src -o target/utils.o
}
```

Then build the project with:

```console
$ nuke build
```

Because we described the relationship between tasks and files using `--files` and `target`,
so **make.nu** is able to build the project incrementally.
