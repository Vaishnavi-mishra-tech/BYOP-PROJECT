import face_recognition
import numpy as np
import os
import pickle
from datetime import datetime

PATH_TO_IMAGES = 'Training_Images'
ENCODING_FILE = 'Encodings/face_encodings.pkl'

def find_encodings():
    """
    Scans the Training_Images folder, generates 128-D face encodings,
    and stores them with the corresponding name.
    """
    image_list = []
    class_names = []
    
  
    if not os.path.exists(PATH_TO_IMAGES):
        print(f"Error: Directory '{PATH_TO_IMAGES}' not found. Create it and add user images.")
        return [], []

    print("Starting encoding generation...")
    
   
    for filename in os.listdir(PATH_TO_IMAGES):
        if filename.endswith(('.jpg', '.jpeg', '.png')):
            # The folder/file name is used as the person's name (e.g., 'John_Doe.jpg')
            name = os.path.splitext(filename)[0]
            
            img_path = os.path.join(PATH_TO_IMAGES, filename)
            img = face_recognition.load_image_file(img_path)
            img_list.append(img)
            class_names.append(name)

    
    encode_list = []
    for img in image_list:
        
        img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB) 
        
        # Check if a face is detected
        face_locations = face_recognition.face_locations(img)
        if face_locations:
            # Generate the 128-D encoding (vector)
            encode = face_recognition.face_encodings(img)[0]
            encode_list.append(encode)
        else:
            print(f"Warning: No face detected in image for {class_names[image_list.index(img)]}. Skipping.")

    print(f"Successfully generated {len(encode_list)} encodings.")
    return encode_list, class_names

def save_encodings(encode_list, class_names):
    """Saves the generated encodings and names to a pickle file."""
    if not os.path.exists('Encodings'):
        os.makedirs('Encodings')

    data = {"encodings": encode_list, "names": class_names}
    with open(ENCODING_FILE, "wb") as f:
        pickle.dump(data, f)
    print(f"Encodings saved successfully to {ENCODING_FILE}")

if __name__ == '__main__':
   
    import cv2 # Import cv2 here since it's used inside find_encodings
    
    known_face_encodings, known_face_names = find_encodings()
    if known_face_encodings:
        save_encodings(known_face_encodings, known_face_names)# khanaklohanaAIML




        import cv2
import numpy as np
import face_recognition
import os
import pickle
from datetime import datetime
import csv
import time

ENCODING_FILE = 'Encodings/face_encodings.pkl'
ATTENDANCE_LOG_DIR = 'Attendance_Logs'

ATTENDANCE_COOLDOWN = 30 

attendance_tracker = {}

def load_encodings():
    """Loads known face encodings and names from the saved file."""
    try:
        with open(ENCODING_FILE, "rb") as f:
            data = pickle.load(f)
            return data["encodings"], data["names"]
    except FileNotFoundError:
        print(f"Error: Encoding file '{ENCODING_FILE}' not found. Run 1_enroll_and_train.py first.")
        return None, None
    except Exception as e:
        print(f"Error loading encodings: {e}")
        return None, None

def mark_attendance(name):
    """Logs the attendance record to a CSV file if not logged recently."""
    now = datetime.now()
    current_time = now.strftime('%H:%M:%S')
    current_date = now.strftime('%Y-%m-%d')
    

    if name in attendance_tracker and (time.time() - attendance_tracker[name]) < ATTENDANCE_COOLDOWN:
        return 
        

    attendance_tracker[name] = time.time()
    
    if not os.path.exists(ATTENDANCE_LOG_DIR):
        os.makedirs(ATTENDANCE_LOG_DIR)
        
    log_file = os.path.join(ATTENDANCE_LOG_DIR, f'{current_date}_Attendance.csv')
    
    
    with open(log_file, 'a+', newline='') as f:
        writer = csv.writer(f)
        
        if os.stat(log_file).st_size == 0:
            writer.writerow(['Name', 'Time', 'Date', 'Status'])
            
        writer.writerow([name, current_time, current_date, 'PRESENT'])
        print(f"ATTENDANCE MARKED: {name} at {current_time}")

def run_attendance_system():
    known_face_encodings, known_face_names = load_encodings()
    if known_face_encodings is None:
        return

    
    cap = cv2.VideoCapture(0)
    
    print("Attendance System Running... Press 'q' to quit.")

    while True:
        success, img = cap.read()
        if not success:
            print("Failed to grab frame.")
            break

        
        img_small = cv2.resize(img, (0, 0), None, 0.25, 0.25)
       
        img_small = cv2.cvtColor(img_small, cv2.COLOR_BGR2RGB)

       
        face_locations = face_recognition.face_locations(img_small)
        face_encodings = face_recognition.face_encodings(img_small, face_locations)

        
        for face_encode, face_loc in zip(face_encodings, face_locations):
            matches = face_recognition.compare_faces(known_face_encodings, face_encode)
            face_distances = face_recognition.face_distance(known_face_encodings, face_encode)

           
            best_match_index = np.argmin(face_distances)
            
            name = "Unknown"
            
           
            if matches[best_match_index] and face_distances[best_match_index] < 0.6: 
                name = known_face_names[best_match_index]
                mark_attendance(name) # Mark attendance for the recognized person

           
            y1, x2, y2, x1 = [int(val * 4) for val in face_loc]
            
            cv2.rectangle(img, (x1, y1), (x2, y2), (0, 255, 0), 2)
            cv2.rectangle(img, (x1, y2 - 35), (x2, y2), (0, 255, 0), cv2.FILLED)
            cv2.putText(img, name, (x1 + 6, y2 - 6), cv2.FONT_HERSHEY_DUPLEX, 1.0, (255, 255, 255), 1)

        cv2.imshow('Face Attendance System', img)
        
       
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv2.destroyAllWindows()

if __name__ == '__main__':
    run_attendance_system()



    import pandas as pd
import os
from datetime import datetime

ATTENDANCE_LOG_DIR = 'Attendance_Logs'


def view_and_export_report():
    """
    Allows the administrator to view and export attendance data.
    """
    
    
    print("--- Administrator Access ---")
    password = input("Enter admin password (demo: 1234): ")
    if password != "1234":
        print("Access Denied.")
        return

   
    log_date_str = input("Enter the date for the report (YYYY-MM-DD, or leave blank for today): ")
    
    if not log_date_str:
        log_date_str = datetime.now().strftime('%Y-%m-%d')
        
    log_file = os.path.join(ATTENDANCE_LOG_DIR, f'{log_date_str}_Attendance.csv')

    try:
        
        df = pd.read_csv(log_file)
        
        
        print(f"\n--- ATTENDANCE REPORT FOR {log_date_str} ---")
        print(df.to_string(index=False))

        export_choice = input("\nDo you want to export this data to Excel? (y/n): ")
        if export_choice.lower() == 'y':
            excel_file = os.path.join(ATTENDANCE_LOG_DIR, f'{log_date_str}_Attendance_Report.xlsx')
            df.to_excel(excel_file, index=False)
            print(f"Report successfully exported to: {excel_file}")
            
    except FileNotFoundError:
        print(f"\nError: Attendance log file for {log_date_str} not found.")
    except Exception as e:
        print(f"An error occurred: {e}")

if __name__ == '__main__':
    view_and_export_report()
