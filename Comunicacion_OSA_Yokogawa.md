# Communication with OSA-Yokogawa using an Ethernet cable

### Matlab programming language - 2024

*Daniel H. Martínez S. - UdeA* \
dhumberto.martinez@udea.edu.co

Yokogawa sources: 
* [Matlab-toolkit](https://tmi.yokogawa.com/solutions/products/oscilloscopes/oscilloscopes-application-software/matlab-wdf-access-toolbox/#Documents-Downloads____downloads_6 "Matlab-Toolkit Yokogawa")
* [Remote-control](https://cdn.tmi.yokogawa.com/1/6057/files/IMAQ6370C-17EN.pdf "Remote-control Yokogawa")
* [User's manual](https://cdn.tmi.yokogawa.com/1/6058/files/IMAQ6370D-01EN.pdf "User's manual Yokogawa")

##  How to control the OSA by Matlab?
There are two ways in order to control the OSA directly from MATLAB. 
* Uses MATLAB Instruments Control Toolbox 
* Uses Yokogawa MATLAB toolkit for OSA (this will be the method explained) 

## Steps to control the OSA by Matlab
1. `Download` and `install` the [Matlab-toolkit](https://tmi.yokogawa.com/solutions/products/oscilloscopes/oscilloscopes-application-software/matlab-wdf-access-toolbox/#Documents-Downloads____downloads_6 "Matlab-Toolkit Yokogawa"). 

2. Read about the `configuration in Matlab` and functions necessary to obtain the traces (*user's manual Matlab-toolkit*).

3. Configure the `communication interface` in the OSA (read **section 3.2** of [Remote-control](https://cdn.tmi.yokogawa.com/1/6057/files/IMAQ6370C-17EN.pdf "Remote-control Yokogawa")).

4. Select **TCP** protocol and verify/note the fixed **IP** in the MANUAL option.

5. Set the laptop's IP with the same submask as the previous step, but the final IP number slightly different.

6. Connect an Ethernet cable between the laptop and the OSA, and ping to verify the recognition of the connection.

7. To save the OSA spectrum/trace on the laptop, run the following code:

```matlab
% :::::: -------         -------  ::::::
% Code modified by D.H. Martínez-Suárez 
% São Paulo  19/07/2022
% :::::: -------         -------  ::::::

%% 
clear all;
close all;
clc

%% Initial communication with the OSA
wire=4;
IP_address = '192.168.0.102'; % OSA fixed IP
Port = '10001';
Username = 'anonymous';
Password = '';
c = ',';
adr = strcat(IP_address,c,Port,c,Username,c,Password)

%% Check for errors
ret = mexOSAComStart(wire,adr);
    if ret ~= 0
        ret = mexOSAGetLastError;
        return;
    end

%% Parameter configuration in the OSA
mexOSASend(':SENS:WAV:CENT 1558nm'); 
mexOSASend(':SENS:WAV:SPAN 100nm'); 
mexOSASend(':SENS:BAND 0.2nm'); 
mexOSASend(':SENS:SENS MID'); 
mexOSASend(':SENS:SWE:SPE 2x'); 
% mexOSASweepSingle; 
% mexOSAComEnd; 


%% Path to save
outDir0 = 'C:\Users\LabFot2\Documents\Medidas Simultaneas\OSA\';
namep = 'Test_spectrum' 

% Time
% datetime('now','TimeZone','local','Format','dd-MM-y HH:mm:ss.ms')
  
% Receive tracedata
[ret, dataX, dataY] = mexOSAGetTrace('TRA');

% data = {datetime('now','Format','dd-MM-yyyy HH:mm:ss.SSS');dataX,dataY}; %,'TimeZone','local'  

data = {dataX;dataX;dataY};

%% Save in .csv format  
s=1
fname = fullfile(outDir0,namep, sprintf('data_%02d.csv', s));
writecell(data,fname)
    

%% receive measurement condition
[ret, cond] = mexOSAGetMeasCond();
if ret ~= 0
    ret = mexOSAGetLastError;
    return;
end

% get XY scales unit
[ret, unitX, ~] = mexOSASendReceive(':UNIT:X?');
if ret ~= 0
    ret = mexOSAGetLastError;
    return;
end

[ret, unitY, ~] = mexOSASendReceive(':DISP:TRAC:Y1:UNIT?');
if ret ~= 0
    ret = mexOSAGetLastError;
    return;
end

%% Close communication port
ret = mexOSAComEnd;
 

%%  Display trace data to a graph
plot(dataX, dataY);
% ylim([-80, 10]);
title('TRA - num2str(s)');
xlabel(strcat('Wavelength [', unitX, ']'));
ylabel(strcat('Power [', unitY, ']'));

```






