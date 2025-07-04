# Aeroguard System

## **Abstract** 

The AeroGuard System is a drone-based safety surveillance solution designed to enhance safety on construction sites through real-time hazard detection. Leveraging advanced machine learning models, specifically YOLO11 and an optimized RT-DETR, the system processes ultra-high-definition 4K video footage to accurately identify safety violations such as workers not wearing helmets, presence of dangerous machinery, and fire hazards. 

Upon detecting a safety violation, the system immediately alerts workers via an onboard alarm and notifies site supervisors by uploading data to existing safety databases. This prompt feedback mechanism ensures swift corrective actions, reducing the risk of accidents and promoting a safer work environment. 

The design process involved evaluating multiple alternatives based on criteria like development cost, technical expertise, and development time. The selected design employs a Raspberry Pi 5 microcontroller with Wi-Fi communication and a high-resolution camera module, striking a balance between performance and cost-effectiveness. 

The software architecture is modular and scalable, featuring components for stream handling, real-time detection, alert management, data uploading, and user interface—all programmed in Python. The user-friendly interface displays real-time detection results and alerts, facilitating easy monitoring and quick decision-making by site supervisors. 

By integrating real-time object detection with immediate alert systems and data storage capabilities, the AeroGuard System aims to significantly improve safety compliance on construction sites. The solution not only enhances operational efficiency but also contributes to a reduction in workplace accidents, aligning with industry safety standards and regulations. 

## **System Interface**

![image051](https://cdn.jsdelivr.net/gh/xiwuqi/image-hosting@master/picgo/202505070059815.png)

You can add the following **Environment Setup** section to your README.md to guide users through configuring and running the project:

------

## Environment Setup

Follow the steps below to set up the environment and run the project:

1. **Clone the Project**

	```bash
	git clone https://github.com/xiwuqi/Software-Part---AeroGuard-Project.git
	cd <your-project-directory>
	```

2. **Create and Activate a Virtual Environment**

	It is recommended to create a virtual environment named `AeroGuard` using `venv`:

	```bash
	python -m venv AeroGuard
	```

	- **On Windows, activate the virtual environment:**

		```bash
		AeroGuard\Scripts\activate
		```

	- **On macOS/Linux, activate the virtual environment:**

		```bash
		source AeroGuard/bin/activate
		```

3. **Install Project Dependencies**

	With the virtual environment activated, install the required packages from `requirements.txt`:

	```bash
	pip install -r requirements.txt
	```

4. **Install PyTorch**

	Visit the [PyTorch website](https://pytorch.org/) and select the installation command that matches your system configuration (operating system, Python version, CUDA version, etc.). Copy the provided `pip install` command and run it in the activated virtual environment. For example:

	```bash
	pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126
	```

	**Note:** Make sure to choose the correct PyTorch version based on your hardware and requirements.

5. **Test the Project**

	The test script for the project is `yolo11-predict-and-track-test.py`. To run the test, execute the following command in the virtual environment:

	```bash
	python yolo11-predict-and-track-test.py
	```

After completing these steps, your project environment will be set up and ready to use for real-time detection and tracking with the AeroGuard System.

------

## **Main Features**

- Extract drone status data, including information such as position and altitude.
- Use object detection and tracking algorithms (e.g., YOLO, RT-DETR) to detect and track drones.
- Determine whether an abnormal situation exists based on the detection and tracking results, and issue an alarm if necessary.

## **Tech Stack**

- Python
- Computer vision algorithms (YOLO, RT-DETR)
- Drone data processing

## **Results of real-time detection**

*Safe condition*

![1738357144](https://cdn.jsdelivr.net/gh/xiwuqi/image-hosting@master/uPic/1738357144.jpg)

*Unsafe condition*

![image049](https://cdn.jsdelivr.net/gh/xiwuqi/image-hosting@master/picgo/202505070059809.png)

