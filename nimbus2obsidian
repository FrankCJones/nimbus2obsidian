import os
import zipfile
import shutil
import html2text
from bs4 import BeautifulSoup

# Define your directories with proper handling for spaces
zip_dir = r'C:\path\to\your\zip files'  # Use raw string notation (r'...') to handle backslashes and spaces
output_dir = r'C:\path\to\output directory'  # Replace with the desired output directory (Obsidian vault)

# Create the output directory if it doesn't exist
os.makedirs(output_dir, exist_ok=True)

# Define the list of extensions of files to keep in assets folders
media_extensions = {
    # Image formats
    '.png', '.jpg', '.jpeg', '.gif', '.bmp', '.tiff', '.svg', '.ico',
    # Audio formats
    '.mp3', '.wav', '.flac', '.aac', '.ogg',
    # Video formats
    '.mp4', '.mov', '.avi', '.mkv', '.webm',
    # Document formats
    '.pdf', '.doc', '.docx', '.odt', '.ppt', '.pptx', '.odp', '.xls', '.xlsx', '.ods', '.csv', '.tsv',
    # Data and script formats
    '.json', '.xml', '.py', '.js', '.sh', '.bat', '.java', '.cpp', '.c',
    # Miscellaneous formats
    '.log', '.yml', '.yaml', '.sql', '.apk', '.dmg',
}

def sanitize_name(name):
    # Replace slashes, backslashes, and exclamation marks with dashes
    return name.replace('!', '-').replace('/', '-').replace('\\', '-')

def html_to_markdown(html_content):
    # Use BeautifulSoup to parse the HTML
    soup = BeautifulSoup(html_content, "html.parser")

    # Remove unnecessary tags that shouldn't be converted to Markdown
    for unwanted in soup(["script", "style", "head", "meta", "link"]):
        unwanted.extract()

    # Convert the cleaned HTML to Markdown using html2text
    converter = html2text.HTML2Text()
    converter.ignore_links = False  # Preserve links
    markdown_content = converter.handle(str(soup))

    return markdown_content

def clean_assets_folder(assets_dir):
    # Recursively remove all files that are not in the media_extensions and clean up empty directories
    for root, dirs, files in os.walk(assets_dir, topdown=False):  # topdown=False allows us to delete files before directories
        for file in files:
            file_path = os.path.join(root, file)
            file_ext = os.path.splitext(file)[1].lower()

            # Remove files that are not in the media_extensions list
            if file_ext not in media_extensions:
                try:
                    os.remove(file_path)
                except Exception as e:
                    print(f"Error removing file {file_path}: {e}")

        # Attempt to remove empty directories
        for dir in dirs:
            dir_path = os.path.join(root, dir)
            try:
                os.rmdir(dir_path)
            except OSError as e:
                print(f"Cannot remove directory {dir_path}: {e}")

def move_assets_and_resolve_conflicts(src, dst, note_file_path):
    if not os.path.exists(dst):
        os.makedirs(dst)
    
    for item in os.listdir(src):
        src_item = os.path.join(src, item)
        dst_item = os.path.join(dst, item)
        
        if os.path.isdir(src_item):
            move_assets_and_resolve_conflicts(src_item, dst_item, note_file_path)
        else:
            # Handle file conflicts by renaming
            if os.path.exists(dst_item):
                base, ext = os.path.splitext(item)
                counter = 1
                new_dst_item = os.path.join(dst, f"{base}_{counter}{ext}")
                while os.path.exists(new_dst_item):
                    counter += 1
                    new_dst_item = os.path.join(dst, f"{base}_{counter}{ext}")
                shutil.move(src_item, new_dst_item)
                # Update the Markdown file to reference the new file name
                with open(note_file_path, 'r', encoding='utf-8') as f:
                    content = f.read()
                content = content.replace(item, f"{base}_{counter}{ext}")
                with open(note_file_path, 'w', encoding='utf-8') as f:
                    f.write(content)
            else:
                shutil.move(src_item, dst_item)

    # Remove the now-empty source directory
    try:
        os.rmdir(src)
    except OSError as e:
        print(f"Cannot remove directory {src}: {e}")

def move_note_files_and_assets(root_dir):
    # Walk through the directory structure
    for root, dirs, files in os.walk(root_dir):
        for dir in dirs:
            note_dir_path = os.path.join(root, dir)
            note_md_path = os.path.join(note_dir_path, f"{dir}.md")
            assets_dir_path = os.path.join(note_dir_path, 'assets')

            # Check if the directory contains the note's .md file
            if os.path.exists(note_md_path):
                # Move the .md file up one level
                new_md_path = os.path.join(root, f"{dir}.md")
                shutil.move(note_md_path, new_md_path)

                # If there's an assets folder, move it and handle conflicts
                if os.path.exists(assets_dir_path):
                    new_assets_dir = os.path.join(root, 'assets')
                    move_assets_and_resolve_conflicts(assets_dir_path, new_assets_dir, new_md_path)

                # Remove the now-empty note directory
                try:
                    os.rmdir(note_dir_path)
                except OSError as e:
                    print(f"Cannot remove directory {note_dir_path}: {e}")

# Step 1: Extract and convert ZIP archives
for root, dirs, files in os.walk(zip_dir):
    for file in files:
        if file.endswith('.zip'):
            try:
                zip_path = os.path.join(root, file)
                note_name = sanitize_name(os.path.splitext(file)[0])

                # Create the directory structure relative to the output directory without flattening
                rel_path = os.path.relpath(root, zip_dir)
                note_output_dir = os.path.join(output_dir, rel_path)
                os.makedirs(note_output_dir, exist_ok=True)

                with zipfile.ZipFile(zip_path, 'r') as zip_ref:
                    zip_ref.extractall(note_output_dir)

                # Rename and convert note.html to note_name.md
                html_file = os.path.join(note_output_dir, 'note.html')
                if os.path.exists(html_file):
                    with open(html_file, 'r', encoding='utf-8') as f:
                        content = f.read()

                    # Convert HTML content to Markdown
                    markdown_content = html_to_markdown(content)

                    md_file = os.path.join(note_output_dir, f'{note_name}.md')
                    with open(md_file, 'w', encoding='utf-8') as f:
                        f.write(markdown_content)

                    os.remove(html_file)  # Optionally remove the original HTML file after conversion

            except zipfile.BadZipFile:
                print(f"Error: '{file}' is not a valid zip file.")
            except FileExistsError as e:
                print(f"Error: {e}")
            except PermissionError:
                print(f"Error: Permission denied while processing '{file}'.")
            except Exception as e:
                print(f"An unexpected error occurred while processing '{file}': {e}")

# Step 2: Move the note files and associated assets up one level in the directory tree
move_note_files_and_assets(output_dir)

print("Conversion complete. Your Obsidian vault is ready.")
