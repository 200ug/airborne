# Shellcode reflective DLL injection in Rust

For a more comprehensive view into the implementation, check out [this blog post](https://dyyni.org/posts/srdi-walkthrough/).

### Project Structure

```shell
.
├── generator           # Shellcode generator (ties together bootstrap, loader, payload, and user data)
├── injector            # PoC injector (CreateRemoteThread)
├── payload             # PoC payload (calc.exe or MessageBoxW based on generator's flag)
├── reflective_loader   # sRDI implementation
└── common              # Common XOR and hashing functions
```

### Features

- ~14 kB reflective loader
- Hashed import names & indirect function calls
- XOR encrypted payload shellcode
- Shuffled and delayed IDT iteration (during IAT patching)

### Usage

The following command compiles the DLLs and executables into `target/release/`:

```shell
$ cargo build --release
```

1. Generate shellcode containing the loader and the payload:

```
Usage: generator.exe [OPTIONS] --loader <LOADER_PATH> --payload <PAYLOAD_PATH> --function <FUNCTION_NAME> --parameter <PARAMETER> --output <OUTPUT_PATH>

Options:
  -l, --loader <LOADER_PATH>      Path to the sRDI loader DLL
  -p, --payload <PAYLOAD_PATH>    Path to the payload DLL
  -f, --function <FUNCTION_NAME>  Name of the function to call in the payload DLL
  -n, --parameter <PARAMETER>     Parameter to pass to the function
  -o, --output <OUTPUT_PATH>      Path to the output file
      --flag <FLAG>               Flag to pass to the loader (by default DllMain is called) [default: 0]
  -h, --help                      Print help
  -V, --version                   Print version
```

2. Inject the created shellcode into target:

```
Usage: poc-injector.exe -p <PROCESS_NAME> -s <SHELLCODE_PATH> -k <KEYFILE_PATH>
```

3. Depending on the flag passed to the generator, either `DllMain` with `DLL_PROCESS_ATTACH` or user function with custom parameter is called:

<div align="center">
    <img src="docs/dllmain-exec.png" alt="Payload's DllMain execution with the default flag (0)" width="90%">
    <img src="docs/userfunction-exec.png" alt="Payload's user defined function execution with the modified flag (1)" width="90%">
</div>

### Credits

- Stephen Fewer ([@stephenfewer](https://github.com/stephenfewer)) for reflective DLL injection
- Nick Landers ([@monoxgas](https://github.com/monoxgas)) for shellcode generator
- [@memN0ps](https://github.com/memN0ps) for bootstrap shellcode
