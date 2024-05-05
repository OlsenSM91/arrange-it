# Arrange.it Web Application

## Overview

Arrange.it is a FastAPI-powered web application designed to streamline the organization of images into teams based on a CSV file. Users can upload a CSV file specifying team assignments and the corresponding image files. The application will automatically sort these images into the designated team folders and generate a downloadable ZIP file containing the organized images.

## Features

- **File Upload**: Users can upload a CSV file along with multiple image files.
- **Image Sorting**: Images are sorted into folders based on team assignments specified in the CSV.
- **Progress Tracking**: Real-time progress updates are provided during file uploads.
- **ZIP Generation**: After sorting, a ZIP file containing the sorted images is generated for download.

## Requirements

- Python 3.8+
- FastAPI
- Uvicorn
- Pandas

This application has been developed and tested on Debian 11.

## Installation

1. **Clone the repository**:
```
git clone https://github.com/yourusername/arrangeit.git
cd arrangeit
```

2. **Set up a virtual environment** (optional, but recommended):
```
python3 -m venv venv
source venv/bin/activate
```

3. **Install dependencies**:
```
pip install fastapi uvicorn pandas
```

4. **Environment Setup**:
Create the necessary directories within the project:
```
mkdir uploads static
```

Add any static files like `favicon.ico` or images to the `static` directory.

5. **Run the application**:
```
uvicorn main:app --host 0.0.0.0 --port 8000
```

## Usage

After starting the server, visit `http://localhost:8000` in your web browser. Follow the instructions on the web page to upload the CSV and image files.

1. **Prepare your CSV file**: Ensure it has at least two columns: `Team` and `Photo`, specifying the team name and the corresponding image file name.
2. **Upload files**: Use the form on the webpage to select your CSV file and the images you wish to sort. Once the files are selected, click the 'Upload Files' button.
3. **Download sorted images**: Once the files have been processed, you will be redirected to download the ZIP file containing the sorted images.

## Acknowledgments

This project was developed to assist photographers in their workflow with applications like PhotoDay and Pixnub. Specifically, it facilitates the use of PhotoDay's capture app and uploading images to their platform. Photographers can then export a CSV with their subjects' data, which is used by this tool to sort the photographed clients into folders based on their class or team for use with Pixnub's EZ Team Builder or EZ Composite Builder.

## Contributing

Contributions to the Arrange.it project are welcome. Please fork the repository and submit pull requests with your suggested changes.

## License

This project is currently unlicensed. You are free to use and distribute it as you see fit until a specific license is applied.
