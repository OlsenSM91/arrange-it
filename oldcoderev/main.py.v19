from fastapi import FastAPI, File, UploadFile
from fastapi.responses import HTMLResponse, JSONResponse
from fastapi.staticfiles import StaticFiles
from datetime import datetime
import uvicorn
import shutil
import os
import pandas as pd
import re

app = FastAPI()

# Define and create a base directory for uploads and static directory for additional files
BASE_DIR = "uploads"
STATIC_DIR = "static"
os.makedirs(BASE_DIR, exist_ok=True)
os.makedirs(STATIC_DIR, exist_ok=True)

# Serve static files from the 'uploads' directory and a general static directory
app.mount("/uploads", StaticFiles(directory=BASE_DIR), name="uploads")
app.mount("/static", StaticFiles(directory=STATIC_DIR), name="static")

@app.get("/", response_class=HTMLResponse)
def read_root():
    return """
    <html>
        <head>
            <title>Arrange.it - File Upload</title>
            <link rel="icon" href="/static/favicon.ico" type="image/x-icon">
            <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
            <script>
                function checkCSVHeaders(file, requiredHeaders, callback) {
                    const reader = new FileReader();
                    reader.onload = function(e) {
                        const text = e.target.result;
                        const headers = text.slice(0, text.indexOf("\\n")).split(",").map(header => header.trim().toLowerCase());
                        const missingHeaders = requiredHeaders.filter(header => !headers.includes(header));
                        callback(missingHeaders);
                    };
                    reader.readAsText(file);
                }

                function uploadFiles(event) {
                    event.preventDefault();
                    var formData = new FormData(document.querySelector('form'));
                    const csvFile = formData.get('csv_file');

                    checkCSVHeaders(csvFile, ['team', 'photo'], function(missingHeaders) {
                        if (missingHeaders.length > 0) {
                            alert('Your CSV file is missing the following required headers: ' + missingHeaders.join(', ') + '. Please check and try again.');
                        } else {
                            $.ajax({
                                xhr: function() {
                                    var xhr = new window.XMLHttpRequest();
                                    xhr.upload.addEventListener("progress", function(evt) {
                                        if (evt.lengthComputable) {
                                            var percentComplete = parseInt((evt.loaded / evt.total) * 100);
                                            document.getElementById('progressBar').style.width = percentComplete + '%';
                                            document.getElementById('progressBar').innerText = percentComplete + '%';
                                        }
                                    }, false);
                                    return xhr;
                                },
                                url: '/upload/',
                                type: 'POST',
                                data: formData,
                                contentType: false,
                                processData: false,
                                success: function(data) {
                                    alert('Files successfully uploaded and processed.');
                                    window.location = '/uploads/' + data.filename; // Redirect to download the zip
                                },
                                error: function() {
                                    alert('Error uploading files.');
                                }
                            });
                        }
                    });
                }
            </script>
            <style>
                body, html {
                    font-family: Arial, sans-serif;
                    background-color: #f4f4f9;
                    color: #333;
                    margin: 0;
                    padding: 0;
                }
                div.main {
                    background-color: white;
                    width: 60%;
                    margin: 2em auto;
                    padding: 2em;
                    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
                }
                h1 {
                    color: #5a5a5a;
                }
                form {
                    margin-top: 20px;
                }
                label {
                    font-weight: bold;
                    display: block;
                    margin: 15px 0 5px;
                }
                input[type="file"], button {
                    padding: 10px;
                    border: 1px solid #ddd;
                    display: block;
                    width: 100%;
                }
                button {
                    background-color: #5c67f2;
                    color: white;
                    border: none;
                    cursor: pointer;
                }
                button:hover {
                    background-color: #5058e5;
                }
                #progressBar {
                    background-color: #4CAF50;
                    color: white;
                    text-align: center;
                    height: 30px;
                    width: 0%;
                    transition: width 0.4s ease-in-out;
                }
                img.logo {
                    display: block;
                    margin: 0 auto 20px auto;
                    max-width: 100%;
                    height: auto;
                }
            </style>
        </head>
        <body>
        <img src="/static/arrangeit-logo.png" alt="Arrange.it Logo" class="logo">
        <div class="main">
            <h1>Arrange.it Organizer</h1>
            <p>Please use the following form to select your CSV that has headers for 'Team' and 'Photo' to create the folders and then the file names you want sorted into those folders. The files will be uploaded to the webserver, and a zip folder with the sorted images will be provided for download.</p>
            <form onsubmit="uploadFiles(event)">
                <label>Select CSV:</label>
                <input type="file" name="csv_file" required><br><br>
                <label>Select Images to Sort:</label>
                <input type="file" name="image_files" multiple required><br><br>
                <button type="submit">Upload Files</button>
            </form>
            <div id="progressBar">0%</div>
        </div>
        </body>
    </html>
    """

@app.post("/upload/")
async def process_files(csv_file: UploadFile = File(...), image_files: list[UploadFile] = File(...)):
    timestamp = datetime.now().strftime("%Y%m%d%H%M%S")
    batch_dir = os.path.join(BASE_DIR, timestamp)
    os.makedirs(batch_dir, exist_ok=True)

    csv_temp_path = os.path.join(batch_dir, csv_file.filename)
    with open(csv_temp_path, 'wb') as f:
        shutil.copyfileobj(csv_file.file, f)

    for image_file in image_files:
        image_path = os.path.join(batch_dir, image_file.filename)
        with open(image_path, 'wb') as f:
            shutil.copyfileobj(image_file.file, f)

    organize_photos_by_team(csv_temp_path, batch_dir)

    zip_path = shutil.make_archive(batch_dir, 'zip', batch_dir)
    zip_filename = f"{timestamp}.zip"

    return JSONResponse(content={"filename": zip_filename})

def organize_photos_by_team(csv_path, image_directory):
    df = pd.read_csv(csv_path)
    df['Team'] = df['Team'].apply(lambda x: re.sub(r'[^a-zA-Z0-9 \n\.]', '', x))
    df['Photo'] = df['Photo'].apply(lambda x: re.sub(r'[^a-zA-Z0-9 \n\.]', '', x))
    for index, row in df.iterrows():
        team_name = row['Team'].strip()
        photo_name = row['Photo'].strip()
        team_directory = os.path.join(image_directory, team_name)
        os.makedirs(team_directory, exist_ok=True)
        source_path = os.path.join(image_directory, photo_name)
        destination_path = os.path.join(team_directory, photo_name)
        if os.path.isfile(source_path):
            shutil.move(source_path, destination_path)

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
