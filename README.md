Prerequisites: English, Windows cmd or Linux bash, understanding of Windows PATH (or the counterpart on Linux), Git (clone, pull, checkout, branch), 50GB SSD, 8 GB memory.

In many situations, you may really enjoy debugging your neo3 smart contract using [neo-express](https://github.com/neo-project/neo-express). However, you may have met some frustrating exceptions raised deep from the source code of neo that you have no idea how to fix them. Besides, you may desire to inspect a running [neo-node](https://github.com/neo-project/neo-node/) instead of its static source codes. 

Follow this guide to set a full neo-cli (3.4.0) on Windows 10 that can be debugged! We will attempt to install the C# environment for neo, and interact with the blockchain using Python. 

Some screenshots in this guide might be in Chinese. 

#### Install Visual Studio 2022 (Community)

Because Visual Studio 2019 does not natively support .NET 6.0, we had better use the latest version. 

Choose .NET desktop development and C++ desktop development, and wipe of some unnecessary components. Here we do not consider Python, but you may also install Python workloads in Visual Studio. The following workloads should be enough (C++ environments are for Python). Be aware that **Windows 10 SDK is needed so that you can compile components of [neo-mamba](https://github.com/CityOfZion/neo-mamba).**

![workload-dotnet](images/workload-dotnet.png)

![workload-cpp](images/workload-cpp.png)

Let Visual Studio Installer download for you. We can just move on. 

#### Install Python 3.8 (and maybe an IDE for Python)

3.9 is also OK, but not 3.7! I suggest downloading it from https://www.python.org/downloads/ .

Ensure that Python executables are added to your PATH, and that the command `pip` can be used. Try this in your local terminal: 

```bash
pip install requests
pip install neo3-boa
```

#### Download codes of neo-cli and other projects

```bash
git clone git@github.com:neo-project/neo-node.git
```

And you may still need to download the [release](https://github.com/neo-project/neo-node/releases) of `neo-cli`. This is just for two `leveldb` dll files inside the released zip file: `Plugins/LevelDBStore.dll` and `libleveldb.dll`. The dlls will be copied into the compiled neo-node later. 

Then you should also 

```bash
git clone git@github.com:neo-project/neo.git
git clone git@github.com:neo-project/neo-vm.git
git clone git@github.com:neo-project/neo-modules.git
git clone git@github.com:CityOfZion/neo-mamba.git
```

Do not forget to consider `git checkout` a proper commit or branch for each repository, so that the versions of all these codes can match. 

#### [Download the **N3 testnet** offline package](https://sync.ngd.network/)

Make sure you are actually downloading N3 testnet 5 (N3T5) full offline package. Unzip it and get the file `chain.0.acc`.

#### Wait for Visual Studio Installation...

The following steps may fail if Visual Studio is not installed.

#### Open `neo-node.sln` with Visual Studio and add projects to it

Add projects `neo`, `neo-vm` and `RpcServer` (in `neo-modules`) to the solution. Add project `neo` as project reference for neo-cli. If there is any compilation error, especially package dependency conflict, consider adding neo-vm as project reference for neo-cli. The package dependencies of neo-cli should be cleared. Optionally and optimally, let `neo-cli` refer to project `RpcServer`, and `RpcServer` refer to `neo`. 

![add-existing-project](images/add-existing-project.png)

![add-project-reference](images/add-project-reference.png)

Let's view the project references of me. `neo-vm` does not need any dependency package or project reference. 

![neo-cli-project-references](images/neo-cli-project-references.png)![neo-project-references](images/neo-project-references.png)![RpcServer-project-references](images/RpcServer-project-references.png)

#### Run neo-cli on testnet at debug mode

Paste the content of `neo-cli/config.testnet.json` into `neo-cli/config.json`, replacing the original `neo-cli/config.json`. This ensures that neo-cli connects to the testnet instead of the mainnet. 

Set neo-cli as the Startup project and debug it. It is likely that the cli would not run properly and throw some exceptions about leveldb. Just copy `libleveldb.dll` to `neo-cli\bin\Debug\net6.0`, and `LevelDBStore.dll` to `neo-cli\bin\Debug\net6.0\Plugins`. 

If neo-cli is started properly, execute `show state` and watch if it is synchronizing blocks. 

#### Sync blocks at insane speed!

Stop neo-cli. Put `chain.0.acc` at `neo-cli\bin\Debug\net6.0`. Add `--noverify` flag in neo-cli debug properties. 

![noverify](images/noverify.png)

Launch neo-cli. 

#### Install RpcServer plugin

If you did not let `neo-cli` refer to `RpcServer`, a simple way here is just to execute `install RpcServer` in `neo-cli` (But remember that the installed `RpcServer` cannot be debugged!)

For a debuggable `RpcServer`, you can replace the installed `neo-cli/bin/Debug/net6.0/Plugins/RpcServer.dll` with that compiled in debugging mode from the project RpcServer in `neo-modules`. Specifically, add project reference `neo` to `RpcServer`, and build `RpcServer` in debug mode. Move `RpcServer.dll` and maybe directory `RpcServer` from `neo-modules/src/RpcServer/bin/Debug/net6.0/` to `neo-cli/bin/Debug/net6.0/Plugins`. Add project reference `RpcServer` for `neo-cli`.  

Now edit `neo-cli/bin/Debug/net6.0/Plugins/RpcServer/config.json` to make sure that your configs matches the testnet. For example, the `network` value should be the same as that in `neo-cli/config.testnet.json`, and meanwhile you probably want to leave `"DisabledMethods": []`. You may also consider larger values for `MaxGasInvoke`, `MaxConcurrentConnections` and `MaxIteratorResultItems`. Watch my config as an example for testnet T5 (with neo-cli 3.4.0):

```json
{
  "PluginConfiguration": {
    "Servers": [
      {
        "Network": 894710606,
        "BindAddress": "0.0.0.0",
        "Port": 16868,
        "SslCert": "",
        "SslCertPassword": "",
        "TrustedAuthorities": [],
        "RpcUser": "",
        "RpcPass": "",
        "MaxGasInvoke": 200,
        "MaxFee": 0.1,
        "MaxConcurrentConnections": 40,
        "MaxIteratorResultItems": 100,
        "DisabledMethods": [],
        "SessionEnabled": true,
        "SessionExpirationTime": 86400
      }
    ]
  }
}
```

Execute `start RpcServer` after RpcServer is equipped with debuggable dlls and correct configs, or just restart neo-cli. 

#### Install neo-mamba

```bash
cd neo-mamba
python setup.py install
```

If `python setup.py install` is deprecated, try `python setup.py build` and then `pip install the-whl-file-that-was-built.whl`

This step involves C language compilation. I cannot really help if there is any error, but if you do meet exceptions, try upgrading your pip and `pip install cmake`.

#### Wait for neo-cli block synchronization

Execute `show state` in neo-cli, and wait for your local blocks to be synced to the latest height. 

![block-sync](images/block-sync.png)

#### Test breakpoints

Congratulations. You are now probably able to inspect the execution of all the assembly instructions in your compiled smart contract. Try adding a breakpoint at `ExecuteInstruction();` in `neo-vm/ExecutionEngine.cs`, near the following codes

```csharp
                    try
                    {
                        ExecuteInstruction();
                    }
                    catch (CatchableException ex) when (Limits.CatchEngineExceptions)
                    {
                        ExecuteThrow(ex.Message);
                    }
```

#### [Data conversion tool for Neo](https://neo.org/converter/index)

#### Test RPC calls

Make sure your neo-cli has been equipped with the plugin `RpcServer`, and that the config (especially `Network`, `MaxGasInvoke`, `MaxFee`, `MaxIteratorResultItems`) is proper. You probably would like to leave `"DisabledMethods": []` to `openwallet` from another process. 

Certainly you can use tools like [Hoppscotch](https://github.com/hoppscotch/hoppscotch) as an HTTP client, but you may also use the client module [neo-fairy-client](https://github.com/Hecate2/neo-fairy-client/) with `git clone git@github.com:Hecate2/neo-fairy-client.git` as a non-official Python RPC client. This package rules out the difficulty from data conversion, and allows you to interact with neo-cli in very natural, detail-irrelevant codes. There are also official RPC clients built in a series of common programming languages, included in official Neo SDKs. Consider [neo-test](https://github.com/ngdenterprise/neo-test) or `NeoRpcClient` in [neo-mamba](https://github.com/CityOfZion/neo-mamba).

Make sure your `invokefunction` calls can break at breakpoints in `neo-vm`. If your RPC calls are well debuggable, consider upgrading your `RpcServer` with [neo-fairy-test](https://github.com/Hecate2/neo-fairy-test/).

#### Compile your contract with .nefdbgnfo!

If your contract is written in Python, the compiler `neo3-boa` should automatically give you an `.nefdbgnfo` file along with the `.nef` binary contract. This file is important, mapping the compiled byte-code neo-vm instructions back to the high-level Python source codes. 

If the contract is in C#, the compiler [nccs](https://www.nuget.org/packages/Neo.Compiler.CSharp/) may not give the `.nefdbgnfo` file. You can compile your contract in this way:

```bash
nccs YourContractCsprojFile.csproj --debug
```

`dotnet restore` and `dotnet build` may be needed for successful compilation (alternatively, click the build and run button of your contract's csproj in your Visual Studio). You can also run your compiler with source codes by `git clone git@github.com:neo-project/neo-devpack-dotnet.git` and by running the project `Neo.Compiler.CSharp` inside, with argument of your contract `.csproj` file and `--debug` flag.

#### DumpNef

https://www.nuget.org/packages/DevHawk.DumpNef/

This is a tool to inspect the disassembly of your `.nef` smart contract. Certainly you can debug with [neo-debugger](https://github.com/neo-project/neo-debugger), but at assembly level with source codes of `neo` and `neo-vm`, you can inspect all the confusing exceptions. For a neo 3.1.0 compatible version, install `DumpNef` with

```bash
dotnet tool install --global DevHawk.DumpNef --version 3.1.9
```

Then, assuring that `.nefdbgnfo` is along with your `.nef` contract file,

```bash
dumpnef YourNefFile.nef
dumpnef YourNefFile.nef > YourNefFile.nef.txt
```

#### Using [Fairy](https://github.com/Hecate2/neo-fairy-test/)

This is something similar to but more powerful than [hardhat](https://github.com/NomicFoundation/hardhat) and [truffle](https://github.com/trufflesuite/truffle). Try placing a Fairy [release](https://github.com/Hecate2/neo-fairy-test/releases) as a plugin of neo-cli (which may not work correctly on different computers...) or building Fairy by yourself along with the source codes required by [Fairy.csproj](https://github.com/Hecate2/neo-fairy-test/blob/master/Fairy.csproj) (which requires a deeper understanding of C# building process). If you have trouble building or resolving project dependencies of Fairy, you may refer to the dependencies of [neo-modules](https://github.com/neo-project/neo-modules). 