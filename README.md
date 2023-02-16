ioq3sim provides an ML-focused version of ioquake3. It does so at the expense of compatibility and security. You should refrain from using this engine for anything besides running simulations and experiments.

# Features
- unbounded execution
- residual memory that persists across map loads
- separate filesystem for experimental data
- qol

# Unbounded execution
The dedicated server of ioq3sim attempts to run at 1000 Hz. Consider using **fixedtime** (5000 is a good starting point), which forces the game module to run # msec of simulation for each server frame. Each instance of ioq3simded should ideally have a physical CPU core running at 100% usage.

# Residual memory
ioq3sim adds a memory zone that persists across server restarts. Its has a default size of **256mb** and can be configured with **com_residualMegs**. This memory zone is exposed to the game module via a set of new system traps:
- **trap_Residual_Alloc(size)** allocate a chunk of memory
- **trap_Residual_Free(\*p)** free a chunk of memory
- **trap_Residual_GetIndex(i)** get a pointer to the i-th allocated chunk (or NULL)
- **trap_Residual_Clear()** reset all residual memory

You will lose all of the memory associated with your simulation when the server restarts, for instance during map (re)loads or when the server automatically cycles (due to time wrapping after ~21 days of game time). That includes pointers to the memory chunks you allocated. To retrieve residual chunks from a previous instance of the server, use **trap_Residual_GetIndex(i)**.

You may for example consider allocating the first chunk (id 0) for a map of your other chunks, i.e.: use the first chunk to hold pointers to your other chunks, e.g. one to your neural network parameters, and another one to your scoring data. After the server restarts, the chunk with id 0 will give you back these pointers - and since residual memory is persistent, these pointers are guaranteed to remain valid.

# Writing & reading experimental data
**fs_datapath** is a new, separate filesystem path for reading & writing experimental data. This path, like **fs_homepath**, is writable. However, it can only be accessed by explicitely putting a **@** prefix in front of file paths:

- **trap_FS_FOpenFile("model.data", FS_WRITE)** will open: /fs_homepath/fs_game/model.data
- **trap_FS_FOpenFile("@model.data", FS_WRITE)** will open: /fs_datapath/fs_game/model.data

You MUST use the **@** prefix for reading files from **fs_datapath**. Otherwise the engine will default to ioquake3's standard filesystem and ignore **fs_datapath**.

A possible setup would be:

- **/usr/local/ioq3sim**	read-only installation files
- **/usr/local/ioq3sim/experiment** read-only experiment setup
- **/home/user/.ioq3sim/experiment** default writable path where ioquake3 outputs non-critical files (game config files, ...)
- **/var/lib/ioq3sim/experiment** path where experimental data is explicitely read & written

In this example, fs_basepath is: /usr/local/ioq3sim; fs_game is: experiment; fs_homepath is: /home/user/.ioq3sim (the default), and fs_datapath is: /var/lib/ioq3sim.

Note that **fs_datapath** supports pk3 files - i.e. the engine also looks inside pk3 files when reading from **fs_datapath**.

# qol
- game scripts support tokens **$0** to **$9**, which are substituted upon calling /**exec**, e.g.: /exec script.cfg arg1 arg2 arg3 replaces $1 $2 $3 in the script with arg1, arg2, arg3.
- **con_numlines** controls the number of notification lines (from the console) to display on the client's HUD.
