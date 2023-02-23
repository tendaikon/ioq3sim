Ioq3sim provides an ML-focused version of ioquake3. It does so at the expense of compatibility and security. You should refrain from using this engine for anything besides running simulations and experiments.

# Features
- unbounded execution
- enhanced performance
- separate filesystem for experimental data
- residual memory that persists across map loads
- qol

# Unbounded execution and performance
Ioq3sim is single-threaded and runs as fast as the system allows it to. Each instance is designed to fully utilize one physical CPU core.

Most of the game performance is spent on 3D geometry - mapping out the world and the objects within it. Ioq3sim enhances performance by modernizing code at chosen hotspots and by using smarter defaults: it tries to optimize the bottlenecks for modern CPUs and disables features like the vanilla bots and anti-cheat systems.

Consider using **fixedtime**. Fixedtime determines how much game time is executed by each server frame (one server frame will run # ms of game time). A value of 0 will synchronize game and server time. Because the server code only performs low-level maintenance, it does not need to be executed at high frequencies. Setting **fixedtime** to a non-zero value can help optimize performance.

Ioq3sim also provides a new Trace() function: **trap_TraceGeometry()**. It can act as a drop-in replacement for trap_Trace(). This new function only considers geometry and ignores all entities. Its application is (thus) contextual - but provides a performance boost over the standard trap_Trace(). Typical entities that clip player movement but are not "geometry" are: other players and map objects such as doors and moving platforms.

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

# Residual memory
ioq3sim adds a memory zone that persists across server restarts. Its has a default size of **256mb** and can be configured with **com_residualMegs**. This memory zone is exposed to the game module via a set of new system traps:
- **trap_Residual_Alloc(size)** allocate a chunk of memory
- **trap_Residual_Free(\*p)** free a chunk of memory
- **trap_Residual_GetIndex(i)** get a pointer to the i-th allocated chunk (or NULL)
- **trap_Residual_Clear()** reset all residual memory

You will lose all of the memory associated with your simulation when the server restarts, for instance during map (re)loads or when the server automatically cycles (due to time wrapping after ~21 days of game time). That includes pointers to the memory chunks you allocated. To retrieve residual chunks from a previous instance of the server, use **trap_Residual_GetIndex(i)**.

You may for example consider allocating the first chunk (id 0) for a map of your other chunks, i.e.: use the first chunk to hold pointers to your other chunks, e.g. one to your neural network parameters, and another one to your scoring data. After the server restarts, the chunk with id 0 will give you back these pointers - and since residual memory is persistent, these pointers are guaranteed to remain valid.

This feature is intended for situations where you require to run agents across multiple maps.

# qol
- game scripts support tokens **$0** to **$9**, which are substituted upon calling /**exec**, e.g.: /exec script.cfg arg1 arg2 arg3 replaces $1 $2 $3 in the script with arg1, arg2, arg3.
- **con_numlines** controls the number of notification lines (from the console) to display on the client's HUD.
