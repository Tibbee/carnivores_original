*This readme will presume you have Build Tools for Visual Studio installed or some other way you obtained the visual c++ compiler (i have it set up as a portable tool). It also presumes you have CMake installed as well.
The default CMakeLists.txt file is set to use MSVC (Microsoft Visual C++) for compiler. A version with compiler set to MinGW will come soon.

To compile the project you will need to go into the Carnivores folder and launch a terminal (cmd.exe) from there, or launch cmd.exe and set the directory where you extracted the Carnviores folder.
Then you can write the following commands.

To build HUNT_3DFX.exe:
cmake . -B Build3DFX -DCMAKE_BUILD_TYPE=Release3DFX

cd Build3DFX

NMake

To build HUNT_D3D.exe:
cmake . -B BuildD3D -DCMAKE_BUILD_TYPE=ReleaseD3D

cd BuildD3D

NMake

To build HUNT_SOFT.exe:
cmake . -B BuildSOFT -DCMAKE_BUILD_TYPE=ReleaseSoft

cd BuildSOFT

NMake

I also recommend using Ninja build tools instead of NMake.
If you installed Ninja into the CMake bin folder, you can also use
-G Ninja command when building the executables and then use Ninja instead of NMake to build the executable.
Ninja is much faster then the default NMake but both will work just fine.
