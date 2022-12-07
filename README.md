ioq3sim provides an ML-focused version of ioquake3. It does so at the expense of compatibility and security. You should abstain from using this engine for anything besides experimenting.

# Features
- unbounded execution for running simulations
- residual memory that persists across map loads
- extra layer of file hierarchy (for multiple runs of the same experiment)
- qol

# Unbounded execution
ioq3sim's dedicated server attempts to run at 1000 Hz. Consider using /fixedtime (a value of 5000 is a good start), which forces the game module to run # msec of simulation for every server frame. Ideally you should have a CPU core running at 100% usage for each instance of ioq3simded.

# Residual memory
ioq3sim adds a memory zone that persists across server restarts. Its size defaults to 256mb and can be configured with com_residualMegs. ioq3sim exposes this memory zone to the game module through a set of new system traps:
- **trap_Residual_Alloc(size)** allocate a chunk of memory
- **trap_Residual_Free(\*p)** free a chunk of memory
- **trap_Residual_GetIndex(i)** get a pointer to the ith allocated chunk
- **trap_Residual_Clear()** reset all residual memory

Once the server restarts you will lose all of your simulation's memory (incl. pointers to the memory you allocated). Use **trap_Residual_GetIndex(i)** to retrieve chunks from a previous instance of the server. A good practice could be to always allocate the first chunk (id 0) for a map of the residual memory, i.e.: use your first call to **trap_Residual_Allocate()** to hold pointers to your other chunks, for example one to your neural network parameters, and a second one to your scoring data. After the server restarts, index 0 will give you back these pointers (and since residual memory is persistent, these are guaranteed to remain valid).

# File hierarchy
fs_subgamedir adds an extra layer of file hierarchy on top of the gamedir. It is meant to separate files and data when you do many runs of the same experiment. Each individual run is given its own filesystem space, e.g.: /exp00 being the gamedir that contains the experiment files, it is further subdivided into /exp00/run00, /exp00/run01, and so on - each of which contains the individual files for one run. Note that the subdir has precedence - i.e.: /exp00/run00/somefile overrides /exp00/somefile.

# qol
- /exec scripts take parameters $0 to $9 which are substituted upon calling /exec with extra parameters. e.g.: /exec script.cfg arg1 arg2 arg3 replaces $1 $2 $3 in the script file with arg1, arg2, arg3.
- con_numlines sets the number of notification lines from the console to display on the client's HUD.
