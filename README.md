# BIU/YU Summer Research Program | Acceleration of Gene Interaction Analysis with Abstract Boolean Networks



Authors: Emily Haller and Nicole Haller


This project was developed starting in summer 2023 as part of the RE:IN gene networks project.
Advised by Prof Hillel Kugler and PhD student Eitan Tannenbaum as part of the BIU-YU Summer Research program.

>*NOTE: this README assumes prior familiarity with [RE:IN](https://www.nature.com/articles/npjsba201610). For an explanation of the technical background (both biological and computational), see abstract [here](https://docs.google.com/document/d/1wbwPwVejVthu6vLt7EooxjrM7Xpp4t5EssfZuHJc_mI/edit?usp=sharing)

The goal of this project is to gradually filter out less-likely gene interactions in .rein files that cause RE:IN to output unrealistic gene network models. This allows users to more efficiently test .rein files for satisfiability and to more accurately study gene interactions.


- I. The first tool works by storing a threshold number that is requested by the user and it uses this threshold number to automatically eliminate some optional interactions from the  original .rein file. These interactions are stored in a new filtered .rein file.


- II. The second tool works by comparing optional interactions in a .rein file and a corresponding excel file which contains numbers indicating how confident we are that an optional interaction will occur. It then prompts the user for the number of new filtered out .rein files they want. Based on this number and the number of optional interactions present in the original .rein file, our program calculates threshold numbers that will output a gradually decreasing (and uniform) number of optional interactions in each new output .rein file it creates. Our program also outputs a text file of commands to input into RE:IN so that they can be tested.





## Projects
----------
A number of new features and improvements were implemented on top of the RE:IN tool. <br /> We built 2 tools: 


### 1 | User Threshold Tool 
----------
Allows the user to enter a threshold number and create a new .rein file  without optional interactions with confidence levels that fall outside the threshold. Also creates a text file of commands to be entered into RE:IN.
- After compiling and running the program **user_threshold.py**, the user is prompted: 
    ```
    Enter a threshold number:
    ```
    - when this threshold number is applied, any optional interactions whose confidence level is outside of the user threshold are eliminated from the output .rein file. 
- Outside the threshold:
    - If the user threshold = positive: eliminate optional interactions whose confidence level is above the threshold or below the negative equivalent of the threshold
    - If the user threshold = negative: eliminate optional interactions whose confidence level is below the threshold or greater than the absolute value of the threshold
        - Explanation: as a result, our tool successfully eliminates a number of less likely optional interactions, based on the number the user chooses.
- Output files:
    - 1. outputFile.rein - a .rein output file that stores only optional interactions within the threshold
        - Location: /Users/{user_home_folder}/outputFile.rein
    - 2. jupyterInput.txt - a text file of commands the user should manually enter into REIN
        - Location: /Users/{user_home_folder}/jupyterInput.txt
        - Example: 
        ```javascript
        let model792 = ReinAPI.LoadFile ”./Science2014/outputFile.rein”
        Model792 |> ReinAPI.DrawBespokeNetworkWithSizeSVG 10000.0
        ReinAPI.CheckAndPrint model792
        ```


### 2 | Automatic Levels Threshold Tool
----------
Allows a user to choose the number of output .rein files to create. It then creates this number of .rein files by automatically calculating and applying different threshold numbers which will divide the optional interactions in the most linear way possible.


- After compiling and running the program **automatic_levels_threshold.py**, the user is prompted: 
    ```
    Enter a value for number of levels you want to test: 
    ```
    - Levels = the number of new output .rein files to create from the original .rein file
    - Based on the number of total optional interactions in the original file, our tool automatically calculates the thresholds which will output the desired number of optional interactions
    - It then distributes a linearly decreasing amount of optional interactions into each respective file using the previously calculated thresholds 
        - Explanation:  This provides for a more uniform distribution of optional interactions and allows for faster testing and analysis by ensuring that each file is unique before testing.
 
- Output files:
    - 1. outputFile{i}.rein - the rein output files that store only optional interactions within their calculated thresholds
        - Location: /Users/{user_home_folder}/outputFile{i}.rein
    - 2. jupyterInput.txt - a text file containing:
            - I. all of the commands (for the number of RE:IN files the user requested) that the user will enter manually into RE:IN 
            - II. the automatic thresholds calculated for each new REIN file
        - Location: /Users/{user_home_folder}/jupyterInput.txt
        - Example: if levels = 3: 
        ```javascript
        let model792 = ReinAPI.LoadFile ”./Science2014/outputFile0.rein”
        Model792 |> ReinAPI.DrawBespokeNetworkWithSizeSVG 10000.0
        ReinAPI.CheckAndPrint model792
        the threshold was: -1e-19

        let model792 = ReinAPI.LoadFile ”./Science2014/outputFile1.rein”
        Model792 |> ReinAPI.DrawBespokeNetworkWithSizeSVG 10000.0
        ReinAPI.CheckAndPrint model792
        the threshold was: 0.496277212615588

        let model792 = ReinAPI.LoadFile ”./Science2014/outputFile2.rein”
        Model792 |> ReinAPI.DrawBespokeNetworkWithSizeSVG 10000.0
        ReinAPI.CheckAndPrint model792
        the threshold was: 0.866548157720856
        ```
>*RULES:
>- If a cell in the excel file doesn’t correspond to its matching .rein file, our program doesn’t consider this interaction. Therefore, it's possible that the original .rein file and the first outputFile0.rein could differ.
>- If the number of optional interactions divided by the number of levels the user enters is not a whole number, our program takes the floor of that number
    <br />- Example: 40 optional interactions, 3 levels, (40/3= floor(13.333) = 13)- for the smallest file, the program will find a threshold number that filters the file so that only 13 optional interactions remain (if impossible, see rule 3 below). Next, the program will look to create a second file which contains 26 optional interactions, and the third which contains 40 optional interactions.
>- If there is no threshold that outputs the exact number of optional interactions we desire, our program looks for the threshold that will output the next closest number of optional interactions (starting with a number lower than the desired number)
    <br />- Example: if we are looking for a threshold that outputs 5 optional interactions and we don't find one, we will look for the threshold that outputs 4 interactions, and if this isn’t found it looks for 6, etc.
