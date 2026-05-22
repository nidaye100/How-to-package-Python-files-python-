# How to Package a Python File into a .exe Executable Program

## 1. Confirm the Environment
Download Python from the official website: https://www.python.org/downloads/. 
During installation, make sure to check the box that says "Add Python to PATH" automatically before proceeding. Afterward, you can open the Command Prompt (cmd) and type the following command to verify if the installation was successful:
python --version

### If Python was installed from other sources (e.g., the Microsoft Store)
You may need to configure the environment variables manually.
1. Open cmd, type `where python` to get the installation path, and copy it. 
   Example: C:\Users\<YourUsername>\AppData\Local\Programs\Python\Python<VersionNumber>
2. Right-click This PC**, select Properties, look for Advanced system settings, and click Environment Variables. Under System variables, locate the Path variable, double-click it, click New, and paste the copied installation path.

### Check if PyInstaller is installed
You can run the following command in cmd to verify:
pip show pyinstaller

If PyInstaller is not installed, install it using:
pip install pyinstaller

Sometimes, even if PyInstaller is installed, the command might not be recognized if its installation path wasn't added to the system PATH. If that happens, you can call PyInstaller via Python instead:
python -m PyInstaller --version

If this runs successfully, PyInstaller is installed correctly but its path is missing from the environment variables. If it fails, you will need to add both the Python installation directory and its Scripts directory to your environment variables.

***Checking the Scripts Directory:***
When Python installs tools like PyInstaller, it places them in the Scripts directory. You can generally find it here:
* If installed via the Microsoft Store, the path typically looks like: C:\Users\<YourUsername>\AppData\Local\Microsoft\WindowsApps\Scripts
* You must add this Scripts directory to your system PATH variable so tools like pyinstaller can be recognized globally in cmd.
* Note: The `where scripts` command is used to locate executable files. It will likely not find the folder because you are looking for a directory, not an executable file.

You can view your current PATH environment variable in cmd by running: echo %PATH%

> Important: You must restart the Command Prompt window after changing environment variables for the updates to take effect.

---

## 2. Package the Python Script

1. Write your Python program and save it as a .py file.
2. In cmd, navigate (cd) into the directory where your script is located, and run:
pyinstaller --onefile --noconsole generate_items.py

--onefile: Bundles everything and all dependencies into a single, standalone .exe file.
--noconsole: Hides the command-line window when running the program (ideal for GUI applications).

Once the process finishes, your executable file will be generated in a newly created folder named dist, titled generate_items.exe.

### Pros and Cons of Packaging
* Pros: The .exe file can be copied and run on other Windows computers instantly, without needing Python installed on those machines.
* Cons: Because it bundles the entire Python runtime and its dependencies, the output file size can be quite large; the .exe file will only run on Windows systems; certain antivirus programs may falsely flag the .exe file as a threat. This is a common issue—if you are distributing your tool, it is highly recommended to include a note about false positives.

---

## 3. Further Optimizations

### 3.1 Using the --icon Parameter to Add a Custom Icon
You can customize the icon of the bundled .exe file by passing an icon path to PyInstaller.

#### 3.1.1 Prepare the Icon File
Ensure you have an icon file in .ico format. If you only have a .png or other image formats, you can use online converter tools to change them into .ico.
* Popular online tools include: ConvertICO (https://convertico.com/) or ICO Convert (https://www.icoconvert.com/).

#### 3.1.2 Specify the Icon During Packaging
Add the --icon parameter when running the PyInstaller command:
pyinstaller --onefile --icon=your_icon.ico your_script.py

--onefile: Packages the script into a single .exe file (optional, depending on your needs).
--icon=your_icon.ico: Specifies the custom icon file you want to apply.

If you have already packaged your .exe and want to change its icon, you must repackage it from scratch using the --icon flag. PyInstaller does not support modifying an existing .exe directly.

Every time you build, PyInstaller generates temporary build configuration files, such as a build folder and a .spec file. If you need to repackage, delete these files first to prevent conflicts.

If the icon does not update, PyInstaller might be caching old settings. You can clear the cache and rebuild using the --clean flag:
pyinstaller --clean --onefile --icon=your_icon.ico your_script.py

If it still doesn't update, try restarting your cmd window or restarting your computer, then package it again.

#### Understanding the Generated Files
* dist/ folder: This is the final output directory containing your completed .exe file. If your script is generate_items.py, you will find generate_items.exe here.
* build/ folder: A temporary folder where PyInstaller stores intermediate files and cache logs during the compilation process. These files are safe to delete once the build is finished.
* generate_items.spec file: A configuration file generated by PyInstaller that contains the layout instructions for building your .exe. If you need advanced customization (like manually bundling extra asset folders), you can edit this .spec file and run PyInstaller directly on it.

Once packaging is done and everything works, you only need to keep the .exe file inside the dist/ folder. You can safely delete the build/ folder and the .spec file.

---

### 3.2 Perfecting File Properties (Metadata)

In Windows, details like the File Version, Product Name, and Copyright are embedded as metadata. For .exe files, this metadata is typically written during the compilation process.

PyInstaller does not offer direct command-line flags to set these fields individually. Instead, you can pass a structured text file using the --version-file parameter.

#### 3.2.1 How to Set File Version Info Using PyInstaller

1. Create a Version Info File: Create a .txt file (often structurally written as an .rc file) and define your metadata fields inside it.
2. Include the Version File: Pass the file using the --version-file parameter when packaging:
pyinstaller --onefile --noconsole --icon=cd.ico --version-file=version.txt generate_items.py

***Best Practices for Writing Metadata***
Adding a File Version, Company Name, Product Name, and Copyright provides clean, professional context for your software. While these properties do not inherently hold standalone legal weight, they are crucial for distribution, version control, and brand clarity. 

1. Version Info (FileVersion and ProductVersion)
   * Purpose: Indicates the current release phase of your app to help developers and users track bugs and updates.
   * Format: Standard convention uses Major.Minor.Patch.Build (e.g., 1.0.0.0), incrementing numbers chronologically with updates.
2. Company Name (CompanyName)
   * Purpose: Shows the organization or individual who created the tool. Use your official business name or personal alias.
3. Product Name (ProductName)
   * Purpose: The commercial title of your software, establishing brand identity. Avoid using trademarked names belonging to other companies.
4. File Description (FileDescription)
   * Purpose: A brief sentence explaining what the .exe does (e.g., "Image Resizing Utility"). Keep it concise and accurate.
5. Copyright (LegalCopyright)
   * Purpose: Declares software ownership. Note: Under international laws, copyright is usually born automatically upon creation, but explicitly writing it deters unauthorized use.
   * Format: Copyright (C) [Year] [Owner Name] (e.g., Copyright (C) 2026 YourCompany). If utilizing open-source libraries, state clearly which parts fall under separate open-source licensing.
6. Trademark (Trademark)
   * Purpose: Identifies registered brand marks. If your product name is legally trademarked, you can append symbols like ™ or ®.
7. License Info (License)
   * Purpose: Clarifies the terms of distribution (e.g., MIT, GPL, or Commercial). This is usually bundled as a separate document with the installer but can be referenced here.

#### Metadata Template File (version.txt)

# UTF-8
VSVersionInfo(
  ffi=FixedFileInfo(
    filevers=(1, 0, 0, 0),
    prodvers=(1, 0, 0, 0),
    mask=0x3f,
    flags=0x0,
    OS=0x4,
    fileType=0x1,
    subtype=0x0,
    date=(0, 0)
  ),
  kids=[
    StringFileInfo(
      [
      StringTable(
        u'040904B0',
        [StringStruct(u'CompanyName', u'YourCompanyName'),
        StringStruct(u'FileDescription', u'Tool for batch generating items.json files in Minecraft Bedrock Edition'),
        StringStruct(u'FileVersion', u'1.0.0.0'),
        StringStruct(u'InternalName', u'generate_items'),
        StringStruct(u'LegalCopyright', u'Copyright (C) 2026 YourCompanyName'),
        StringStruct(u'OriginalFilename', u'generate_items.exe'),
        StringStruct(u'ProductName', u'ndy100'),
        StringStruct(u'ProductVersion', u'1.0.0.0')])
      ]
    ),
    VarFileInfo([VarStruct(u'Translation', [1033, 1200])])
  ]
)

***Key Technical Details to Remember:***
* VSVersionInfo Structure: This script uses highly strict nested structures (FixedFileInfo, StringFileInfo, StringTable, etc.). Make sure every bracket and comma matches the format exactly, or PyInstaller's parser will crash.
* Version Number Format: Versions in FixedFileInfo must be written as tuples of integers separated by commas—not strings (e.g., (1, 0, 0, 0)).
* Language and Code Page (Translation): The value 1033 specifies English (United States), and 1200 enables Unicode compatibility, ensuring characters render cleanly across varying systems.
