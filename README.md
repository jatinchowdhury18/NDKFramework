# NDKFramework
Repository of the NDKFramework as showcased at Audio Developer Conference 2019

## Purpose

Analog circuit emulations can be achieved using State-space processing. However, prototyping in this domain can be cumbersome, as its computation is non-modular and for each circuit iteration, the necessary matrices have to be computed anew. The NDKFramework reduces this modularity issue by deriving the matrices from the netlist specifications of the circuit in the application LTSPice. A set of Matlab scripts and extensions to the JUCE framework allow for offline-debugging and real-time simulation of analog circuits.

## Dependencies

The following applications are needed to use the NDKFramework: 
* [LtSpice](https://www.analog.com/en/design-center/design-tools-and-calculators/ltspice-simulator.html)
* [MATLAB](https://www.mathworks.com/products/matlab.html). Optional but highly recommended for offline-debugging.
* [JUCE](https://juce.com)

As well as the following libraries: 
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page).  Use `brew install eigen` if you're using brew on macOS
* [nlohmann-json]( https://github.com/nlohmann/json) Use `brew install nlohmann-json` if you're using brew on macOS
* [NumPy](https://numpy.org/). Refer to your Python package manager of choice to install it.

## Usage

The NDKFramework facilitates the following workflow: 
1. Design your circuit in LTSpice. Abide to the specified [naming conventions](https://github.com/dstrub18/NDKFramework/blob/master/README.md#naming-conventions) of the components, otherwise, they will not be parsed. Also refer to the list of [currently supported components](https://github.com/dstrub18/NDKFramework/blob/master/README.md#currently-supported-components).
2. Save the netlist of your circuit as follows:
    * Right-click on the editing panel -> View -> SPICE Netlist
    * CMD-S to save the netlist
3. Export the netlist to a .json file as follows:
    * Open the terminal
    * Type `python3 <relative-path-to-Main.py> <relative-path-to-netlist-file> <relative-desired-output-path> `
4. Open the .json file in Matlab using the template files in this repository to 
    * As of now, you have to **explicitly** specify the equations for the nonlinear components you used in your circuit. Refer to the           example circuits.
5. Open the NDKCircuitTemplate in the Projucer. in the `prepareToPlay()` method, specify the .json path in the stateSpaceProcessor constructor.
    * `stateSpaceProcessor = std::make_unique<StateSpaceProcessor>("path-to-file.json", sampleRate); `
6. Build the project.

## Currently supported components
* Voltage Sources
* Resistor
* Potentiometers
* Capacitor
* Inductor
* 1N914 Diode (Shockley equation)
* 2N2222 NPN Transistor (Ebers-Moll equation)
* 12AX7 Triode (Equations according [Dempwolf](http://recherche.ircam.fr/pub/dafx11/Papers/76_e.pdf))
* Operational amplifier (Only linear behaviour)


## Naming conventions
* ` # ` is any number (multiple digits are allowed)
* N_FROM and N_TO are the nodes the component is connected from / to
* All component groups are required to be **zero-indexed**
* `VIN`is the entry point for the audio samples later on
```
Resistor       := R#<NAME>    N_FROM N_TO    VALUE
Capacitor      := C#<NAME>    N_FROM N_TO    VALUE
Inductor       := L#<NAME>    N_FROM N_TO    VALUE
Voltage Input  := VIN#<NAME>  N_FROM N_TO
Voltage Supply := VCC#<NAME>  N_FROM N_TO    <"DC">   VALUE
```
* As potentiometers are not natively supported in LTSpice, two resistors with the maximum and minimum value have to be created instead. 

```
Potentiometer Top := RT#<NAME> N_FROM N_TO VALUE
Potentiometer Btm := RB#<NAME> N_FROM N_TO 

Voltage Out := VOUT#<NAME> 
Ground      := 0

Diode    := D#<NAME>    N_FROM N_TO    MODEL
NPN      := Q#<NAME>    NC NB NE NS    MODEL
Triode   := XU#<NAME>   NA NG NC       MODEL
Opamp    := XUOPA<NAME> NI I  VCC   VEE   OUT   MODEL
```
## Potentiometer handling in C++
* As of now, the only way to let the user change the circuit behavior is via potentiometers. These can be accessed by creating in a slider in the `PluginEditor` and call the function `StateSpaceProcessor::updatePotValues` with the corresponding index for each potentiometer.
