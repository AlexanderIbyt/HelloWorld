# Installing ibyt (Linux)

ibyt is a systems language between assembler and C. The compiler turns a `Main.ibyt` into a native Linux program (`MyProgram`) or into a bootable image for bare-metal machines.

## What you need

- A 64-bit Linux (x86-64)
- The ibyt compiler (`ibytc`) and the `library` folder, from the download below
- Nothing else for normal programs: the compiler writes the executable itself. For some programs it falls back to `nasm` and `ld`; if it asks for them:

| Distribution | Command |
|---|---|
| Debian / Ubuntu / Mint | `sudo apt install nasm binutils` |
| Fedora / RHEL | `sudo dnf install nasm binutils` |
| Arch / Manjaro | `sudo pacman -S nasm binutils` |
| openSUSE | `sudo zypper install nasm binutils` |

To try bare-metal images you also want QEMU (`sudo apt install qemu-system-x86`).

## 1. Download

Download the latest release from the **Releases** page of this repository (`ibyt-<version>.zip`) and unpack it:

    unzip ibyt-<version>.zip

You get two things: the compiler `ibytc` and the folder `library`.

    chmod +x ibytc        # only if the file is not executable yet

## 2. Your first program

Make a project folder and copy the compiler and the library into it:

    mkdir HelloWorld && cd HelloWorld
    cp ../ibyt-<version>/ibytc .
    cp -r ../ibyt-<version>/library .

Create a file named `Main.ibyt` (the name is required):

    import <Basic.ibyt>

    Start{
        prepare $pBase: "Hello World!\n" call Basic.Print(solid base)
    }

Run the compiler in this folder:

    ./ibytc

It prints `Success! ... 'MyProgram'`. Start your program:

    ./MyProgram

Output: `Hello World!`

## 3. A bootable program (no operating system)

Start the file with `ibyt_bare` and import `BasicBare` instead of `Basic`:

    ibyt_bare
    import <BasicBare.ibyt>

    Start{
        prepare $pBase: "Hello from bare metal!\n" call BasicBare.Print(solid base)
        exit
    }

`./ibytc` writes `MyProgram.img`. Run it in QEMU:

    qemu-system-x86_64 -drive format=raw,file=MyProgram.img -serial stdio

Or write it to a USB stick (**this erases the stick; check the device name with `lsblk`**):

    sudo dd if=MyProgram.img of=/dev/sdX bs=1M status=progress

The PC must start in BIOS (Legacy/CSM) mode; pure UEFI machines do not boot it.

## Problems

| Message | What to do |
|---|---|
| `Main.ibyt` not found | The compiler must be started in the folder that contains `Main.ibyt`. |
| Module `Basic.ibyt` not found | The `library` folder must sit next to `Main.ibyt`. |
| `The register list has a different number of entries...` | A call needs the argument list from the library description (`library/Name.md`), e.g. `Basic.Print(solid base)`. |
| `nasm`/`ld` not found | Install them as shown above. |
