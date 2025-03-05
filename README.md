# Markdown-to-EPUB-Automator-Workflow
在 macOS 上使用 Automator + Pandoc 将 Markdown 文件（.md）转换为 EPUB 文件（.epub），方便上传到readwise

Use Automator + Pandoc on macOS to convert Markdown files (.md) to EPUB files (.epub) for easy uploading to Readwise.    
* * *

下面的示例将演示如何在 macOS 上使用 Automator + Pandoc 将 Markdown 文件（.md）转换为 EPUB 文件（.epub）。整个流程分为以下几个步骤：

* * *

一、安装 Pandoc
-----------

1.  **Homebrew 安装（推荐）**  
    打开终端，输入以下命令安装 Pandoc：
    
    ```bash
    brew install pandoc
    ```
    
    如果没有安装 Homebrew，请先按照 [Homebrew 官方文档](https://brew.sh/) 的指引进行安装。
    
2.  **直接下载可执行文件**  
    如果不想使用 Homebrew，可以从 [Pandoc 官方发布页面](https://github.com/jgm/pandoc/releases) 下载安装包或可执行文件，并将其放置到系统的可执行路径（如 `/usr/local/bin` 或 `/opt/homebrew/bin`）下。
    

安装完成后，终端中输入：

```bash
pandoc -v
```

即可查看安装版本，确认安装成功。

* * *

二、创建 Automator 脚本（Quick Action 或服务）
-----------------------------------

1.  **打开 Automator**  
    在「应用程序」文件夹或 Launchpad 中找到并打开 **Automator**。
    
2.  **新建文档**
    
    *   在 Automator 弹出的「新建文稿」对话框中，选择「**快速操作**（Quick Action）」或「**服务**」(视系统版本而定)，然后点击「选择」。
    *   如果没有自动弹出「新建文稿」对话框，可以在菜单栏中选择「文件 > 新建...」，再选择「快速操作」或「服务」。
3.  **配置工作流属性**
    
    *   在右上方「接收以下类型的内容」(或「服务收到所选的」) 中，选择「**文件或文件夹**」或者「Finder 中的文件」。
    *   建议将「工作流程接受当前」(或「服务流程用」) 设为「**Finder**」应用，以便在 Finder 中右键文件时使用。
    *   「替换所选内容」保持默认不勾选即可。
4.  **添加“运行 Shell 脚本”操作**
    
    *   在左侧的动作栏里搜索「运行 Shell 脚本」(Run Shell Script)，将其拖拽到右侧工作区中。
    *   在「运行 Shell 脚本」面板内：
        *   Shell 选择：`/bin/bash` 或 `/bin/zsh` 均可，这里以 `/bin/bash` 为例。
        *   传递输入：选「**作为参数**」(或「to arguments」)。
5.  **编写 Shell 脚本**  
    在「运行 Shell 脚本」文本框内粘贴以下脚本代码：
    
    ```bash
    #!/bin/bash
    
    # 如果你的 pandoc 不在默认 PATH 中，可以使用以下形式指定 pandoc 的完整路径
    # PANDOC_PATH="/usr/local/bin/pandoc"
    # 或者 /opt/homebrew/bin/pandoc 等
    # 如果你可以直接调用 pandoc，就不需要下面这行
    PANDOC_PATH="pandoc"
    
    for f in "$@"
    do
        # 如果文件后缀是.md，将其替换为.epub
        # 例如 ~/Documents/test.md -> ~/Documents/test.epub
        base="${f%.md}"
        # 如果你的文件可能有其他后缀 (.markdown 等)，可自行调整
        
        "$PANDOC_PATH" "$f" -o "$base.epub"
    done
    ```
    
6.  **保存 Quick Action**
    
    *   在 Automator 菜单栏选择「文件 > 存储...」(或按 Command+S)。
    *   为此工作流取一个易识别的名称，例如「**Markdown 转 EPUB**」。
    *   位置保持默认即可（会自动保存到 `~/Library/Services/` 目录）。

* * *

三、使用方法
------

1.  **在 Finder 中右键文件**
    
    *   打开 Finder，定位到需要转换的 `.md` 文件。
    *   右键（或按住 Control 点击）你想转换的 Markdown 文件。
    *   在弹出的菜单中找到「服务」(或「快速操作」) 选项，然后选择你刚才创建的「**Markdown 转 EPUB**」。
2.  **等待转换完成**
    
    *   Automator 会调用 Pandoc 脚本对所选的 Markdown 文件进行转换。
    *   脚本结束后，你将在相同文件夹下得到同名的 `.epub` 文件。

* * *

四、常见问题及优化
---------

1.  **Pandoc 路径问题**
    
    *   如果在脚本中调用 `pandoc` 报错「command not found」，则说明脚本执行环境中 PATH 配置不包含 Pandoc 的安装位置。
    *   你可以：
        *   在脚本中使用 Pandoc 的绝对路径，例如：
            
            ```bash
            /usr/local/bin/pandoc "$f" -o "$base.epub"
            ```
            
        *   或者在脚本开始处手动声明 PATH：
            
            ```bash
            export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"
            ```
            
2.  **批量转换**
    
    *   如果一次性选中多个 `.md` 文件，同样可以批量转换。脚本中 `for f in "$@"` 会遍历所有选中的文件。
3.  **后缀判断**
    
    *   如果你的 Markdown 文件的后缀不一定是 `.md`（可能是 `.markdown` 等），可以在脚本中作更灵活的处理，比如：
        
        ```bash
        extension="${f##*.}"
        base="${f%.*}"
        if [ "$extension" = "md" ] || [ "$extension" = "markdown" ]; then
            "$PANDOC_PATH" "$f" -o "$base.epub"
        fi
        ```
        
4.  **Pandoc 选项**
    
    *   你可以在 Pandoc 命令后添加更多选项，例如指定目录、元数据文件、样式等。
    *   例如，指定 TOC（目录）和封面：
        
        ```bash
        pandoc "$f" --toc --epub-cover-image=cover.jpg -o "$base.epub"
        ```
        
    *   具体可参考 Pandoc 官方文档。

* * *

1\. Install Pandoc
------------------

### a. Using Homebrew (Recommended)

1.  Open the Terminal.
2.  Run the command:
    
    ```bash
    brew install pandoc
    ```
    
3.  If you haven’t installed Homebrew yet, follow the instructions on the [Homebrew website](https://brew.sh/).

### b. Downloading the Executable

Alternatively, download the installer or executable from the [Pandoc releases page](https://github.com/jgm/pandoc/releases) and place it in a directory included in your system’s PATH (such as `/usr/local/bin` or `/opt/homebrew/bin`).

After installation, confirm by running:

```bash
pandoc -v
```

This should display the installed Pandoc version.

* * *

2\. Create an Automator Quick Action (or Service)
-------------------------------------------------

### a. Open Automator

*   Find **Automator** in your Applications folder or via Launchpad and launch it.

### b. Create a New Document

*   When prompted with the “New Document” dialog, choose **Quick Action** (or **Service**, depending on your macOS version), then click **Choose**.
*   (If the dialog does not appear, go to **File > New** and select **Quick Action** or **Service**.)

### c. Configure the Workflow

*   In the upper-right area, set “Workflow receives current” (or “Service receives selected”) to **files or folders** (or **Finder items**).
*   Optionally, set the application to **Finder** to allow conversion via the context menu.
*   Leave “Replace Selected Items” unchecked.

### d. Add the “Run Shell Script” Action

*   In the left sidebar, search for **Run Shell Script** and drag it into the workflow area.
*   In the action settings:
    *   Choose the shell (for example, `/bin/bash` or `/bin/zsh`). Here, we use `/bin/bash`.
    *   Set “Pass input:” to **as arguments**.

* * *

3\. Write the Shell Script
--------------------------

Paste the following script into the **Run Shell Script** text area:

```bash
#!/bin/bash

# If pandoc is not in the default PATH, specify the full path:
# PANDOC_PATH="/usr/local/bin/pandoc"
# If pandoc is accessible directly, you can simply use:
PANDOC_PATH="pandoc"

for f in "$@"
do
    # Remove the .md extension to get the base file name.
    # For example: ~/Documents/test.md becomes ~/Documents/test
    base="${f%.md}"
    
    # Convert the Markdown file to EPUB
    "$PANDOC_PATH" "$f" -o "$base.epub"
done
```

#### Notes:

*   The script checks each selected file (using the loop `for f in "$@"`) and converts files ending with `.md` to an EPUB by stripping the extension and appending `.epub`.
*   If your Markdown files have different extensions (like `.markdown`), you may adjust the script. For example:
    
    ```bash
    extension="${f##*.}"
    base="${f%.*}"
    if [ "$extension" = "md" ] || [ "$extension" = "markdown" ]; then
        "$PANDOC_PATH" "$f" -o "$base.epub"
    fi
    ```
    

* * *

4\. Save and Use the Quick Action
---------------------------------

### a. Save the Workflow

*   Choose **File > Save** (or press Command+S).
*   Give your workflow a name such as **Markdown to EPUB**.
*   The workflow will be saved in `~/Library/Services/` (or the default location for Quick Actions).

### b. Using the Workflow

*   Open **Finder** and navigate to a Markdown file (.md).
*   Right-click (or Control-click) the file.
*   In the context menu, choose **Quick Actions** (or **Services**) and then select **Markdown to EPUB**.
*   The Automator action will run, converting your Markdown file to an EPUB in the same folder.

* * *

5\. Additional Tips
-------------------

*   **Pandoc Path Issues:**  
    If you receive an error stating “command not found” for `pandoc`, ensure the PATH is set correctly in the script. You can export the PATH at the beginning of the script, for example:
    
    ```bash
    export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"
    ```
    
*   **Batch Conversion:**  
    The script handles multiple files at once because it loops over all passed files.
*   **Pandoc Options:**  
    You can add additional options to the Pandoc command. For example, to include a table of contents and a cover image:
    
    ```bash
    pandoc "$f" --toc --epub-cover-image=cover.jpg -o "$base.epub"
    ```
    
    Check the Pandoc Manual for more options.

* * *

By following these steps, you now have an Automator Quick Action on your Mac that allows you to easily convert Markdown files to EPUB format by right-clicking the file in Finder. Enjoy your streamlined workflow!

* * *
