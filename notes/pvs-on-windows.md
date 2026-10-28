# Using PVS on Windows (updated)

PVS works only on Unix based operating systems, meaning it won't normally work
on Windows. We can get around this by installing and running PVS on Windows
Subsystem for Linux (WSL).

1. Install WSL (this guide assumes you install Ubuntu). See [these installation instructions](https://learn.microsoft.com/en-us/windows/wsl/install#install-wsl-command).
2. Launch WSL (`wsl` in a terminal) and run  these commands:

    ```sh
    sudo apt update && sudo apt upgrade && sudo apt install sbcl
    ```

3. Clone [nasalib](https://github.com/nasa/pvslib) and set the environment variable `PVS_LIBRARY_PATH` in your shell.

    Ex.

    ```sh
    git clone https://github.com/nasa/pvslib.git ~/PVS/
    # (edit this path )
    echo 'export PVS_LIBRARY_PATH="~/PVS/nasalib"' >> ~/.bashrc
    ```

4. Follow [these instructions](...) to install Visual Studio Code from within WSL
5. Run `code` from inside a WSL terminal
6. Install VSCode-PVS (see instructions on [the GitHub page](https://github.com/nasa/vscode-pvs#installation-instructions---pvsv80))

    > [!NOTE]
    > Do not install VSCode-PVS from the extension store. Follow the
    > instructions in the README in the link above.

7. Complete the default installation process when prompted by the extension.
   This may take a while. Provide the path to nasalib (Step 3) if prompted.
