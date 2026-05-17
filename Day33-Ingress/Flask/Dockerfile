# Use a official image as a parent images
FROM python:3.11-slim

# Set the working directory in the container
WORKDIR /app

# Copy the dependencies file in the workdir /app
COPY requirements.txt .

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Copy the content of the local src directory to the working directory
COPY . .

# Run app.py when the container launches
CMD ["python", "app.py"]