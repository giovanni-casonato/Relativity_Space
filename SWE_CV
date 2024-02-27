import cv2
import numpy as np
import csv

def image_cropping(img):

    img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    # Find the brightest point in the image and crop around it
    _, max_val, _, max_loc = cv2.minMaxLoc(img)
    columns_range = (max_loc[0] + 75, max_loc[0] + 300)
    rows_range = (max_loc[1] - 50, max_loc[1] + 100)

    img = img[rows_range[0]:rows_range[1], columns_range[0]:columns_range[1]]

    return img

def measure_weld_width(img):

    #Blur and Get edges
    cropped_img = image_cropping(img)
    img = cv2.medianBlur(cropped_img, 7)
    edges = cv2.Canny(img, 450, 500, apertureSize=5)

    # Use Hough Line Transform to detect lines
    lines = cv2.HoughLines(edges, 1, np.pi/180, threshold=1)

    # Draw the detected lines on the original image based on slope < 0.01
    img_with_lines = cropped_img.copy()

    line1_points = None
    line2_points = None

    for line in lines:
        rho, theta = line[0]
        a = np.cos(theta)
        b = np.sin(theta)

        if b != 0:
            slope = -a / b 

            # Check if the absolute value of the slope is within the threshold
            if abs(slope) < 0.01:
                x0 = int(a * rho)
                y0 = int(b * rho)
                x1 = int(x0 + 1000 * (-b))
                y1 = int(y0 + 1000 * (a))
                x2 = int(x0 - 1000 * (-b))
                y2 = int(y0 - 1000 * (a))

                cv2.line(img_with_lines, (x1, y1), (x2, y2), (255, 255, 255), 1)

                if line1_points is None:
                    line1_points = [(x1, y1), (x2, y2)]
                else:
                    line2_points = [(x1, y1), (x2, y2)]
                    break 

    # Calculate the width and plot it
    width = np.sqrt((line2_points[0][0] - line1_points[0][0])**2 + (line2_points[0][1] - line1_points[0][1])**2)
    cv2.line(img_with_lines, (line1_points[0][0]+ 1100, line1_points[0][1]), (line2_points[0][0]+1100, line2_points[0][1]), (255, 255, 255), 2)

    # #   DELETE BELOW TO GET RID OF PLOTTING
    # cv2.imshow('Modified Image', img_with_lines)
    # cv2.waitKey(0)
    # cv2.destroyAllWindows()
    # #   DELETE ABOVE

    return width

def process_video(video_path, output_csv):
    cap = cv2.VideoCapture(video_path)

    # Open output CSV file
    with open(output_csv, 'w', newline='') as csvfile:
        csv_writer = csv.writer(csvfile)
        csv_writer.writerow(['Time (seconds)', 'Bead Width (pixels)'])

        # Process each frame
        while True:
            ret, frame = cap.read()

            if not ret:
                break

            # Get width
            bead_width = measure_weld_width(frame)

            if bead_width > 10:
                # Write time and bead width to CSV
                time = cap.get(cv2.CAP_PROP_POS_MSEC) / 1000.0
                csv_writer.writerow([time, bead_width])

        # Release video capture and close CSV file
        cap.release()
        csvfile.close()

# Example usage
video_file = r"C:\Users\Giovanni\Desktop\Desktop\Relativity Space\sw_candidates_proj0\videos\weld.mp4"
output_csv_file = "weld_width_measurements.csv"

process_video(video_file, output_csv_file)
