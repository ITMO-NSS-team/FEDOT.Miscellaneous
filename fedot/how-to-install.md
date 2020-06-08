## FEDOT Framework installation guide

## How to install FEDOT Framework to your system:

* **Step 1**. *Download FEDOT Framework*.
  * First of all, you need to copy or 'download' FEDOT Framework to your personal computer. You can do it directly using the button          'clone or download' (red square) or you can install 'Jetbrains toolbox' and using the "clone in Pycharm" button (blue square), 
    which will open the files you need directly in the Pycharm project. 
  * For more details, see a picture below.
    * ![Step 1](img/img-tutorial/1_step.png)
* **Step 2**. *Creating VirtualEnv in Pycharm project*.
  * Next, you need to create virtual enviroment in your Pycharm project. To do this, go through the following chain: 
    *'File-Settings-Project Interprenter-Add new'*. 
  * For more details, see a picture below.
    * ![Step 2](img/img-tutorial/2_step.PNG)
  * After you have created a virtual environment, you should install the libraries necessary for the FEDOT framework to work. 
    In order to do this, go to the terminal console (blue square) and run the following command *pip install -r requirements.txt* 
    (red square). 
  * For more details, see a picture below.
    * ![Step 3](img/img-tutorial/3_step.png)
* **Step 3**. *Manually installing libraries*.
  * In order to use the framework visualization tools, you will additionally need to manually install the graphviz library.
    * Download "graphviz-2.38.msi" from https://graphviz.gitlab.io/_pages/Download/Download_windows.html
    * Execute the "graphviz-2.38.msi" file
    * Add the graphviz bin folder to the PATH (blue square) system environment variable (Example: "C:\Graphviz2.38\bin") (red square) 
  * For more details, see a picture below.
    * ![Step 4](img/img-tutorial/4_step.png)
