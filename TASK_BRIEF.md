Hi Rakib,

Practical Task: AI Computer Vision Developer
📌 Project Title:
Camera-Based Product Detection System (YOLO Object Detection)


🎯 Task Objective:
Build a simple AI-based product detection system that can detect common grocery items from images or video frames using a YOLO model or similar object detection approach.


🧠 Task Requirements:
1️⃣ Dataset Handling
Use any small dataset OR collect sample images of:

Rice
Oil bottle
Soap / detergent
Packet items (any grocery type)
👉 You can use:

Roboflow dataset
Public datasets
Or manually collected images

2️⃣ Model Training / Setup
Use YOLOv5 or YOLOv8
Train or fine-tune a model (pretrained allowed)
Must detect at least 2–3 object classes

3️⃣ Object Detection Output
When an image is given, model should return:

{
"detections": [
{
"class": "rice",
"confidence": 0.92
},
{
"class": "oil",
"confidence": 0.88
}
]
}


4️⃣ API Development (Mandatory)
Build a simple API using FastAPI / Flask

Endpoint:
POST /detect

Input:
Image upload
Output:
JSON detection result

5️⃣ Optional (Bonus)
Real-time webcam detection
Video stream processing
Docker container setup

📦 Deliverables:
Candidate must submit:

1️⃣ GitHub Repository
Full source code
Training / inference scripts
API code

2️⃣ Model File
Trained .pt or .onnx file

3️⃣ Demo Video (VERY IMPORTANT)
🎥 Candidate must record a video showing:

Code explanation
Model running
Image upload → detection result
API working (Postman / browser)
👉 Video length: 3–10 minutes


4️⃣ README File
Must include:

How to run project
How to test API
Dataset info
Model details

🧠 Evaluation Criteria:
We will evaluate based on:

Model accuracy & logic
Code structure
API implementation
Real-time capability understanding
Problem solving ability
Clean project architecture

⚠️ Important Notes:
Copy-paste projects will be rejected
Must understand YOLO / object detection basics
Must be able to explain code in video
Pretrained model allowed but customization required

🚀 Submission Format:
Send:

GitHub link
Demo video link (Google Drive / YouTube unlisted)
Short explanation of approach

🎯 Final Goal:
We want to see if the candidate can build:

👉 A working AI detection module + API integration
This is the core AI part of our Camera Inventory System.  📦 Recommended Datasets
1️⃣ 🛒 Retail Product Dataset (Best for your use case)
🔹 SKU-110K Dataset
Very popular for retail shelf detection
Thousands of supermarket shelf images
Good for dense object detection (many products in one image)
👉 Use case:

Shop shelf detection
Counting products

🔹 Grocery Products Dataset (Roboflow)
Contains labeled grocery items (rice, oil, drinks, etc.)
Easy to use with YOLO format
👉 Best for:

Quick MVP training
Product-level detection

2️⃣ 🧴 Product-specific datasets
🔹 Retail Product Checkout Dataset
Focus on checkout counter items
Clean labeled objects
👉 Good for:

Simple classification + detection

3️⃣ 🧠 General Object Detection (backup option)
🔹 COCO Dataset
80 general classes
Not retail-specific but useful for pretraining
👉 Use for:

Transfer learning (VERY IMPORTANT)

4️⃣ 🧪 Easy Starter Dataset (Recommended for your junior test)
🔹 Roboflow Universe Datasets
👉 https://universe.roboflow.com

Search:

“grocery detection”
“supermarket products”
“retail shelf dataset”
👉 Why this is best:

Already YOLO formatted
Ready to train in minutes
No heavy preprocessing needed

🚀 Best Strategy for Your Project
👉 Don’t rely on one dataset

Use this combo:

COCO (pretraining)
Grocery / SKU dataset (fine-tuning)
Your own shop images (final real system)

💡 Simple Recommendation (Important)
For your hiring task:

👉 Tell candidates:

“You can use COCO + any Roboflow grocery dataset”
This ensures:
✔ Faster submission
✔ Real skill check
✔ No dataset barrier


🎯 Final Answer
👉 Best dataset for your project:

SKU-110K (best retail dataset)
Roboflow Grocery datasets (best for MVP)
COCO (for base model training)   
read and tell me what to do|

read and tell me what to do