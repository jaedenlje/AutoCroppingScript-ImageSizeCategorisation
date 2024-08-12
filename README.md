
# Auto Cropping Script (Image Size Categorisation)
## Description
This project provides a script to process a CSV file containing image annotations, crop images based on bounding box (bbox) information, and save the cropped images in a structured directory hierarchy based on class names and pixel sizes.

## Table of Contents
- [Installation](#installation)
- [Usage](#usage)
- [License](#license)

## Installation
Install [PyCharm Community Edition](https://www.jetbrains.com/pycharm/download/?section=windows)


1. Create a new project:

![Screenshot (5)](https://github.com/user-attachments/assets/505ebcc0-a23f-41de-8e75-bd82759452ce)


2. Open settings and select "Python Intepreter" under your project. Click on the '+' sign and search for "opencv-python" to install the OpenCV package:

![Screenshot (2)](https://github.com/user-attachments/assets/1bc46e42-2b96-404d-8a32-f3347c3db87d)
![Screenshot (6)](https://github.com/user-attachments/assets/a913794f-e252-47f4-84ee-5599aa880fb0)
![Screenshot (7)](https://github.com/user-attachments/assets/18c56eba-8351-470a-b263-fdf4a6077608)

3. Create a new Python file:

![Screenshot (4)](https://github.com/user-attachments/assets/7344ef74-ca51-4d8e-be36-91933edf2906)

## Usage
To use the script, you need to provide the path to the CSV file containing image annotations, the folder containing the images, and the output folder where the cropped images will be saved.

    1. Update the csv_file_path, image_folder_path and output_folder_path variables in the script with the appropriate paths.
    2. Run the script:

### Example
Here is an example of how to use the script:

    import os
    import cv2


    def crop_bbox(image_path, bbox):
        try:
            image = cv2.imread(image_path)
            if image is None:
                print(f"Error reading image: {image_path}")
                return None
            xmin, ymin, xmax, ymax = bbox
            if xmin >= xmax or ymin >= ymax:
                print(f"Invalid bbox coordinates for image {image_path}: {bbox}")
                return None
            cropped_image = image[ymin:ymax, xmin:xmax]
            return cropped_image
        except Exception as e:
            print(f"Error cropping image {image_path}: {str(e)}")
            return None


    def process_csv_file(csv_file_path, image_folder_path, output_folder_path):
        with open(csv_file_path, 'r') as csv_file:
            next(csv_file)  # Skip header
            unsorted_folder = os.path.join(output_folder_path, 'Unsorted')
            if not os.path.exists(unsorted_folder):
                os.makedirs(unsorted_folder)
            counter = 1  # Initialize counter
            for csv_line in csv_file:
                parts = csv_line.strip().split(',')
                image_filename = parts[0]
                class_name = parts[1]
                bbox = tuple(map(int, parts[2:6]))
                image_path = os.path.join(image_folder_path, image_filename)
                if not os.path.exists(image_path) or not os.path.isfile(image_path):
                    print(f"Image file not found or not a file: {image_path}")
                    continue
                class_folder = os.path.join(output_folder_path, class_name)
                if not os.path.exists(class_folder):
                    os.makedirs(class_folder)
                cropped_image = crop_bbox(image_path, bbox)
                if cropped_image is not None:
                    height, width, _ = cropped_image.shape
                    min_length = min(height, width)
                    if height > width:
                        selected_length = width
                    else:
                        selected_length = height
                    if selected_length < 20:
                        size_folder = os.path.join(class_folder, 'very_small')
                    elif selected_length < 30:
                        size_folder = os.path.join(class_folder, 'small')
                    elif selected_length < 50:
                        size_folder = os.path.join(class_folder, 'medium')
                    elif selected_length < 70:
                        size_folder = os.path.join(class_folder, 'large')
                    else:
                        size_folder = os.path.join(class_folder, 'very large')
                    if not os.path.exists(size_folder):
                        os.makedirs(size_folder)
                    output_image_path = os.path.join(size_folder, f"{image_filename.rsplit('.', 1)[0]}_{counter}.{image_filename.rsplit('.', 1)[1]}")
                    if cv2.imwrite(output_image_path, cropped_image):
                        print(f"Image saved successfully: {output_image_path} ({counter})")
                        counter += 1  # Increment counter
                    else:
                        unsorted_image_path = os.path.join(unsorted_folder, image_filename)
                        if cv2.imwrite(unsorted_image_path, cropped_image):
                            print(f"Image saved in Unsorted folder: {unsorted_image_path}")
                        else:
                            print(f"Error saving image: {image_path}")
                else:
                    print(f"Error cropping image: {image_path}")


    # Example usage
    csv_file_path = "/path/to/your/csv/file"
    image_folder_path = "/path/to/your/image/folder"
    output_folder_path = "/path/to/your/output/folder"
    process_csv_file(csv_file_path, image_folder_path, output_folder_path)

## License
This project is licensed under the [MIT License](https://www.mit.edu/~amini/LICENSE.md).



