# SFND_Radar_Target_Generation_and_Detection
<img src="./Overview.jpg"/>
Main file: Radar_Target_Generation_and_Detection.m

2D CFAR Steps:

* Determine the number of Training cells for each dimension. Similarly, pick the number of guard cells.
%%Training cell dimensions were chosen iteratively.

    %Select the number of Training Cells in both the dimensions.
    Td = 2; %Doppler training cell dim
    Tr = 10; %Range training cell dim

Same for the Guard cell dimension. Numbers are random, but because Nd < Nr => Gd < Gr

    %Select the number of Guard Cells in both dimensions around the Cell under 
    %test (CUT) for accurate estimation
    Gd = 2; %Doppler guard cell dim
    Gr = 5; %Range guard cell dim

Offset was chosen in an approximate fashion. On average, values of RDM were around 24. Max value was 40. 
    
    % offset the threshold by SNR value in dB
    snr_offset = 6;
 
In order to suppress the non-thresholded cells at the edges I just created a matrix of 0s with size of RDM. Cells where RDM values were exceeding threshold were assigned the value of 1.

    RDM_filtered = zeros(size(RDM));
    
* Slide the cell under test across the complete matrix. Make sure the CUT has margin for Training and Guard cells from the edges.
* For every iteration sum the signal level within all the training cells. To sum convert the value from logarithmic to linear using db2pow function.
* Average the summed values for all of the training cells used. After averaging convert it back to logarithmic using pow2db.
* Further add the offset to it to determine the threshold.
* Next, compare the signal under CUT against this threshold.
* If the CUT level > threshold assign it a value of 1, else equate it to 0.
```
    
    for i = Tr+Gr+1:(Nr/2)-(Gr+Tr)
        for j = Td+Gd+1:Nd-(Gd+Td)
            train_cell = RDM(i - Gr - Tr : i + Gr + Tr, j - Gd - Td : j + Gd + Td);
            train_cell = db2pow(train_cell);
            train_cell(i - Gr : i + Gr, j - Gd : j + Gd) = 0;
            train_avg = sum(sum(train_cell)) / N_Train; %average
            train_avg = pow2db(train_avg);
            threshold = train_avg + snr_offset;
            if (RDM(i,j) > threshold)
                RDM_filtered(i,j) = 1;
            end
        end
    end




    
