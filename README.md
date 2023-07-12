# Configure of Vscode debug
## Build tool chain
``` shell
# install tool chain
sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu 
# newer qemu
sudo apt-get remove qemu-system-misc
sudo apt-get install qemu-system-misc=1:4.2-3ubuntu6
```
## GDB
``` shell
make qemu-gdb
# based on hint, add .gdbinit under /home/dg
vim .gdbinit
# add below config to .gdbinit
add-auto-load-safe-path /home/dg/xv6-riscv/.gdbinit

gdb-multiarch kernel/kernel
```
## Combine Vscode with GDB
实现vscode调试xv6，配置时需要注意.vscode和要编译项目不在一个目录下，因此要调整tasks中cwd和launch中program。
launch.json：
``` json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "xv6debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/xv6-riscv/kernel/kernel",
            "stopAtEntry": true,
            "cwd": "${workspaceFolder}",
            "miDebuggerServerAddress": "127.0.0.1:26000", 
            "miDebuggerPath": "/usr/bin/gdb-multiarch", 
            "MIMode": "gdb",
            "preLaunchTask": "xv6build"
        }
    ]
}
```
tasks.json:
problemMatcher中指定了从Now run 'gdb' in another window这个位置开始追踪（有时间可以学一下problemMatcher）
``` json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "xv6build",
            "type": "shell",
            "isBackground": true,
            "command": "make qemu-gdb",
            "options": {
                "cwd":"${workspaceFolder}/xv6-riscv"
            },
            "problemMatcher": [
                {
                    "pattern": [
                        {
                            "regexp": ".",
                            "file": 1,
                            "location": 2,
                            "message": 3
                        }
                    ],
                    "background": {
                        "beginsPattern": ".*Now run 'gdb' in another window.",
                        "endsPattern": "."
                    }
                }
            ]
        }
    ]
}
```
实现各个文件之间正确的跳转
``` shell
# bear for project dependency
sudo apt-get install bear
make clean&&bear -- make qemu
# move compile_commands.json to .vscode
mv compile_commands.json ../.vscode/
vim c_cpp_properties.json
```
c_cpp_properties.json：
``` json
{
    "configurations": [
        {
            "name": "Linux",
            "compileCommands": "${workspaceFolder}/.vscode/compile_commands.json"
        }
    ],
    "version": 4
}
```