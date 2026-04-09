Introduction:

Attendance management is a fundamental and critical administrative task in educational institutions and corporate environments. This process depends on manual methods like calling out names or circulating paper sign-in sheets. These systems are generally time-consuming, disrupt the flow of lectures or work, and are highly inaccurate and unethical practices, most notably proxy attendance (where one person marks another absent individual as present). So, to address these inefficiencies, this project proposes an advanced solution to the same: “the Face Recognition-Based Attendance System.”

This system uses computer vision and machine learning technology to recognise people in a live video stream and automatically mark their presence. Video input is captured, human faces are detected, the identity of the detected face is determined by comparing it with a database of pre-registered facial encodings, and then attendance is then recorded with a timestamp.

This system's successful implementation shows the practical application of algorithms like Haar Cascades or Histogram of Orientated Gradients for detection and FaceNet for extremely accurate face recognition. The primary tools used in the project's Python construction are the OpenCV and face recognition libraries, which ensures a dependable and efficient solution.

The overall goal is to provide a reliable, efficient, and non-intrusive mechanism for attendance record-keeping, saving valuable institutional time and improving the attendance data.

Scope requirements:
1. In-scope:
•	Real-Time Attendance Marking: The system will capture live video from a standard webcam, detect faces, and automatically mark the attendance of recognized, pre-registered users with a date and time stamp.
•	User Enrolment and Training: An administrator interface will allow for capturing multiple facial images per user and generating 128-dimensional face encodings for the recognition model's database.
•	Database Management: The system will store user profile data (Name, ID, Encodings) and manage the daily attendance log records.
•	Report Generation: The system will allow to view, filter, and export attendance reports for administrative use. 

2. Out-of-scope:
•	Advanced Anti-Spoofing: This project scope does not include complex anti-spoofing mechanisms (e.g., blink detection or 3D mapping) to verify a live person v/s a photograph.
•	Cloud Deployment: The system will be implemented as a local, standalone applications and will not be deployed to a cloud platform for web-based access.
•	Multi-Camera Support: The implementation is limited to processing video input from a single, given camera source at any given time.

Target Users:
User role: Primary Function/Interaction
Administrator (Faculty/Supervisor):	Responsible for system setup, user enrollment (FR1), managing the attendance process, and generating data reports (FR3).
End User (Student/Employee):	The subject whose attendance is being automatically tracked and recorded via facial recognition (FR2).
