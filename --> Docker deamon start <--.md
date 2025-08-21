--> Docker deamon start <--

sudo systemctl start docker

sudo systemctl status docker

sudo systemctl stop docker

sudo systemctl restart docker



--> Upload bulk files to min.io via api <--

for file in test_data/*; do
  echo "Uploading: $file"
  curl -F "file=@$file" http://localhost:5000/buckets/test-bucket/upload
  echo ""
done


--> Verify upload <--

echo "=== Uploaded Files ==="
curl http://localhost:5000/buckets/test-bucket/objects





--> URL Structure with Single Bucket + Folder Organization <--

warrant-storage/
├── media/
│   ├── images/
│   │   ├── 2024/08/19/user-123-avatar.jpg
│   │   └── 2024/08/19/product-456-main.jpg
│   ├── documents/
│   │   └── 2024/08/19/contract-789.pdf
│   └── temp/
│       └── 2024/08/19/upload-abc123.tmp
├── avatars/
│   └── users/
│       └── user-123-profile.jpg
└── backups/
    └── 2024/08/19/
        └── database-backup.sql



--> Direct MinIO URLs (if public) <--

https://your-minio-server.com/warrant-storage/media/images/2024/08/19/user-123-avatar.jpg

--> Presigned URLs (secure, temporary) <--

https://your-minio-server.com/warrant-storage/media/images/2024/08/19/user-123-avatar.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=...&X-Amz-Date=...&X-Amz-Expires=600&X-Amz-SignedHeaders=host&X-Amz-Signature=...



--> Upload to Django (replaces your direct MinIO upload) <--

curl -F "file=@test_data/data.json" \
     -F "folder=documents" \
     http://localhost:8000/api/media/upload/


--> Upload with specific category <--

curl -F "file=@test_data/image.jpg" \
     -F "folder=media/products" \
     http://localhost:8000/api/media/upload/


--> Bulk Upload Script for Django <--

#!/bin/bash
# Updated script for Django API
for file in test_data/*; do
  echo "Uploading: $file to Django API"
  
  # Determine folder based on file type
  if [[ "$file" == *.jpg ]] || [[ "$file" == *.png ]]; then
    folder="media/images"
  elif [[ "$file" == *.pdf ]]; then
    folder="documents"
  elif [[ "$file" == *.json ]]; then
    folder="data"
  else
    folder="misc"
  fi
  
  curl -F "file=@$file" \
       -F "folder=$folder" \
       http://localhost:8000/api/media/upload/
  echo ""
done


--> Storage Service Configuration <--

class StorageService:
    def __init__(self):
        self.bucket_name = settings.MINIO_BUCKET  # warrant-storage
    
    def upload_file(self, file, folder='media', subfolder=None):
        """
        Upload file with folder structure
        folder: media, documents, avatars, temp
        subfolder: images, products, users, etc.
        """
        # Generate path: media/images/2024/08/19/filename.jpg
        today = datetime.now()
        date_path = f"{today.year}/{today.month:02d}/{today.day:02d}"
        
        if subfolder:
            object_key = f"{folder}/{subfolder}/{date_path}/{file.name}"
        else:
            object_key = f"{folder}/{date_path}/{file.name}"
        
        # Upload to: warrant-storage/media/images/2024/08/19/filename.jpg
        return self.client.upload_fileobj(
            file, 
            self.bucket_name, 
            object_key
        )
    
    def get_file_url(self, object_key, expires_in=600):
        """Generate presigned URL for file access"""
        return self.client.generate_presigned_url(
            'get_object',
            Params={'Bucket': self.bucket_name, 'Key': object_key},
            ExpiresIn=expires_in
        )

        

--> API Response Examples <--

Upload Response:
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "filename": "data.json",
  "content_type": "application/json",
  "size": 1024,
  "object_key": "documents/2024/08/19/data.json",
  "folder": "documents",
  "upload_url": null,
  "access_url": "https://your-minio-server.com/warrant-storage/documents/2024/08/19/data.json?X-Amz-..."
}

Get File Response:
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "presigned_url": "https://your-minio-server.com/warrant-storage/documents/2024/08/19/data.json?X-Amz-Algorithm=AWS4-HMAC-SHA256&...",
  "expires_at": "2024-08-19T11:05:00Z",
  "filename": "data.json"
}


--> Migration from Direct MinIO <--

Your current direct MinIO uploads can be replaced:
# OLD: Direct to MinIO
curl -F "file=@test_data/data.json" http://localhost:5000/buckets/test-bucket/upload

# NEW: Through Django API  
curl -F "file=@test_data/data.json" -F "folder=documents" http://localhost:8000/api/media/upload/

