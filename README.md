# K-Bot Optical Denisty (OD) Sensor
K-Bot and K-Bot Auto are custom sensor devices for measuring the OD of anaerobic bacteria directly within growth vessel. Both use a 3D printed tube holder with an LED and photosensor on opposing side. The photosensor outputs a voltage value that is converted to the $OD_{600}$ of the sample by a species specific calibration equation. K-Bot Auto is an advanced version of K-Bot that is integrated with other lab equipment for autonomous multi-day experiments.

## Project Libraries
Each versions necessary Python libraries can be installed via respective requirements.txt file. requirements.txt in root of this repository installs all libraries for both K-Bot and K-Bot Auto. 
**<p align="center"> pip install -r requirements.txt </p>**  


## K-Bot
Portable 3D-printed bench top optical density (OD) sensor. Utilizes LabJack U3 data acquisition device (DAQ) to power an ~ 600nm LED and collect analog voltage values from a TEMT600 photosensor within the K-Bot device. Analog voltages are converted to OD via species specific calibration curves. DAQ is connected to laptop running terminal based  application (RunPhotosensor), that uses Export_csv.py file in backend. RunPhotosensor operation consists of user specifying bacteria species to be measured and then collecting as many samples as needed. Avaliable bacterial species to be measured are determined by Equations.json file that contains the species specific calibration equations to convert voltage to OD.

## K-Bot Auto
K-Bot Auto is discrete K-bot sensor integrated with oscillating shaker and IR LED panel for autonomous multiday and multisample (up to 4) experiments. K-bot Auto uses Seeed XIAO ESP32S3 microcontroller with built-in wifi and Bluetooth capabilities for modulating component timing and for real time streaming of OD values to online ThingSpeak database.  

Uses LED panel for photosynthetic growth of bacteria, oscillating shaker to maintain sample homogenity, IoT relay to modulate timing, and Hall effect sensor to determine osciallator position (only want to take readings at apex of shaker oscillation for greatest precision). 

Experiment operating conditions specified by setExperimentalParameters.py and uploaded to Thingspeak database. Microcontroller polls a voltage value every tinterval seconds until Max_Time or Max_Num is reached and then will blink LED 4 times to signal experiment termination. Experimental data can be downloaded as easy-to-read CSV file via downloadData.py or through ThingSpeak website.

ThingsSpeak Online Database Field Variables:
Field 1 = Uploaded OD values  
Field 2 = Max_Num (number of measurements)  
Field 3 = Max_Time (run time)  
Field 4 = tinterval (measurement time interval)

## Code description
Optical density is acquired by Export_csv.py (directory K-Bot). A LabJack U3 is instantiated as dev = u3.U3(); if instantiation fails the script substitutes a constant 1 V test voltage, a debug path not used for reported data. FindLEDPinNumber() identifies the digital line driving the LED by setting FIO6 high, reading the photosensor on analog input AIN0, and assigning that line if the reading exceeds 0.5 V, repeating for FIO7. The line is held high for the session, illuminating the LED continuously, and driven low on exit. For each sample, the operator declares a species, validated by checkValidName() against the keys of Equations.json, a calibration dictionary that an unrecognized entry prompts the user to extend. measurement() polls AIN0 over a fixed window at a fixed interval (duration = 1 s, interval = 0.1 s, nominally duration / interval = 10 readings) and reduces the readings to an arithmetic mean, average_V. ConvtoOD() evaluates the species expression against average_V in a namespace exposing only math and Voltage. All expressions take the form OD = m · ln(V) + b, the logarithmic form expected for transmitted intensity under Beer-Lambert attenuation, with (m, b) fitted by regression of benchtop spectrophotometer readings against ln(V): R. palustris (-0.68, 0.8628), R. gelatinosus CBS (-0.631, 0.2489), R. rubrum (-0.586, 0.2872). Each OD, a datetime.now() timestamp and the species are appended to a list, written on exit to a CSV under the header ["Time", "Sample Type", "OD Value"] in append mode so repeated sessions accumulate in one file. Calibrations are reached only through load_equations() and save_equations(), so the calibration set is data rather than code; misc/getEq.py exposes the same functions standalone. ContinuosReadings.py is a hardware diagnostic that prints raw AIN0 voltage every 2 s with no calibration or logging, and was not used for reported data.
