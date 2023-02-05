ioq3sim provides an ML-focused version of ioquake3. It does so at the expense of compatibility and security. You should abstain from using this engine for anything besides experimenting.

# Features
- unbounded execution for running simulations
- residual memory that persists across map loads
- separate filesystem path for experimental data (fs_datapath)
- qol

# Unbounded execution
ioq3sim's dedicated server attempts to run at 1000 Hz. Consider using **fixedtime** (a value of 5000 is a good start), which forces the game module to run # msec of simulation for every server frame. Ideally you should have a CPU core running at 100% usage for each instance of ioq3simded.

# Residual memory
ioq3sim adds a memory zone that persists across server restarts. Its size defaults to **256mb** and can be configured with **com_residualMegs**. ioq3sim exposes this memory zone to the game module through a set of new system traps:
- **trap_Residual_Alloc(size)** allocate a chunk of memory
- **trap_Residual_Free(\*p)** free a chunk of memory
- **trap_Residual_GetIndex(i)** get a pointer to the ith allocated chunk
- **trap_Residual_Clear()** reset all residual memory

Once the server restarts you will lose all of your simulation's memory (incl. pointers to the memory you allocated). Use **trap_Residual_GetIndex(i)** to retrieve chunks from a previous instance of the server. A good practice could be to always allocate the first chunk (id 0) for a map of the residual memory, i.e.: use your first call to **trap_Residual_Allocate()** to hold pointers to your other chunks, for example one to your neural network parameters, and a second one to your scoring data. After the server restarts, index 0 will give you back these pointers (and since residual memory is persistent, these are guaranteed to remain valid).

# Writing & reading experimental data
ioq3sim adds a separate filesystem path for reading & writing experimental data: **fs_datapath**. This path is writable, just like **fs_homepath**. It can however only be accessed by explicitely using a @ prefix in front of file paths:

- **trap_FS_FOpenFile("model.data", FS_WRITE)** will open: /fs_homepath/fs_game/model.data
- **trap_FS_FOpenFile("@model.data", FS_WRITE)** will open: /fs_datapath/fs_game/model.data

You MUST use the @ prefix for reading files from **fs_datapath**. Otherwise the engine will default to ioquake3's standard filesystem hierarchy and ignore **fs_datapath**.

A good setup could be:

- **/usr/local/ioq3sim**	read-only installation files
- **/usr/local/ioq3sim/experiment** read-only experiment setup
- **/home/user/.ioq3sim/experiment** default writable path where ioquake3 outputs non-critical files (local config files, ...)
- **/var/lib/ioq3sim/experiment** path where experimental data is explicitely read & written

In this setup, **fs_basepath** is **/usr/local/ioq3sim**, **fs_game** is **experiment**, **fs_homepath** is **/home/user/.ioq3sim** (the default), and **fs_datapath** is **/var/lib/ioq3sim**.

Note that **fs_datapath** supports pk3 files - i.e. the engine looks inside pk3 files as well when reading from **fs_datapath**.

# qol
- /**exec** scripts take parameters $0 to $9 which are substituted upon calling /**exec** with extra parameters. e.g.: /exec script.cfg arg1 arg2 arg3 replaces $1 $2 $3 in the script file with arg1, arg2, arg3.
- **con_numlines** sets the number of notification lines from the console to display on the client's HUD.
