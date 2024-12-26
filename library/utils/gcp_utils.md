# gcp utils

### Sync dataset from a GCP bucket and folder to a local folder

- Code
    
    ```python
    import os
    from google.cloud import storage
    import shutil
    from tqdm import tqdm
    
    def sync_bucket_to_local(bucket_name, bucket_folder, local_folder):
        """ Sync dataset from a GCP bucket and folder to a local folder"""
        storage_client = storage.Client()
        bucket = storage_client.get_bucket(bucket_name)
    
        # list all blobs in the bucket folder
        blobs = bucket.list_blobs(prefix=bucket_folder)
        print(f"Syncing {len(list(blobs))} files from the {bucket_name}/{bucket_folder} bucket folder to {local_folder}...")
        
        for blob in tqdm(blobs):
            # construct the local file path
            local_file_path = os.path.join(local_folder, os.path.relpath(blob.name, start=bucket_folder))
    
            if os.path.exists(local_file_path):
                # skip files that already exist in the local folder
                continue
    
            # create the local directory if it doesn't exist
            os.makedirs(os.path.dirname(local_file_path), exist_ok=True)
            if blob.name[-1] == "/":
                continue  # skip directories
            
            # download the blob to the local file path
            with open(local_file_path, "wb") as file:
                blob.download_to_file(file)
    
        # delete local files that don't exist in the bucket
        print(f"Looking for files in {local_folder} that do not exist in {bucket_name}/{bucket_folder}...")
        deleted_count = 0
        for root, dirs, files in os.walk(local_folder):
            for file in files:
                local_file_path = os.path.join(root, file)
                relative_file_path = os.path.relpath(local_file_path, start=local_folder)
                bucket_file_path = os.path.join(bucket_folder, relative_file_path)
    
                if not bucket.blob(bucket_file_path).exists():
                    os.remove(local_file_path)
                    print(f"Deleted: {relative_file_path}")
                    deleted_count += 1
    
        print(f"{deleted_count} files were deleted from {local_folder}.")
    ```
    

### Tunnel to my deepam-minda-torch-general

```bash
PROJECT_ID='data-science-test-343606'
export ZONE='us-west1-b'
export INSTANCE_NAME='deepam-minda-general'
gcloud compute ssh --project $PROJECT_ID --zone $ZONE \
$INSTANCE_NAME -- -L 8080:localhost:8080

```


### upload data directly to bucket (without local-file storing)

- this chat explains the code and various content types: https://chatgpt.com/share/82c915d8-5299-496d-bee2-2ef43b8b44b4

```python
import pickle
from google.cloud import storage

def upload_list_as_binary_to_gcs(bucket_name, destination_blob_name, data_list):
    storage_client = storage.Client()
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(destination_blob_name)
    
    binary_data = pickle.dumps(data_list)
    blob.upload_from_string(binary_data, content_type='application/octet-stream')

# Example usage:
bucket_name = "your_bucket_name"
destination_blob_name = "path/to/destination/blob.pkl"
large_list = [i for i in range(1000000)]  # Example large list

upload_list_as_binary_to_gcs(bucket_name, destination_blob_name, large_list)

```