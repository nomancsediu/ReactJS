# File Upload

File uploads are everywhere. Profile pictures, document attachments, cover images. I used to think file upload was just an input tag. Then I tried to add progress bars, previews, and drag and drop. It gets complicated fast. Let me walk you through it step by step.

## Basic File Input

The simplest way to upload a file:

```jsx
function FileUpload() {
  const [file, setFile] = useState(null);

  const handleChange = (e) => {
    const selected = e.target.files[0];
    if (selected) setFile(selected);
  };

  const handleUpload = async () => {
    const formData = new FormData();
    formData.append("file", file);

    await fetch("/api/upload", {
      method: "POST",
      body: formData,
    });
  };

  return (
    <div>
      <input type="file" onChange={handleChange} accept="image/*" />
      <button onClick={handleUpload} disabled={!file}>Upload</button>
    </div>
  );
}
```

Do not set `Content-Type` to `multipart/form-data` yourself. The browser sets it automatically with the correct boundary when you use `FormData`.

## Image Preview

Users want to see what they are uploading before they commit:

```jsx
function ImageUploadWithPreview() {
  const [file, setFile] = useState(null);
  const [preview, setPreview] = useState(null);

  const handleChange = (e) => {
    const selected = e.target.files[0];
    if (!selected) return;
    setFile(selected);
    setPreview(URL.createObjectURL(selected));
  };

  // Clean up the object URL to avoid memory leaks
  useEffect(() => {
    return () => {
      if (preview) URL.revokeObjectURL(preview);
    };
  }, [preview]);

  return (
    <div>
      <input type="file" onChange={handleChange} accept="image/*" />
      {preview && <img src={preview} alt="Preview" className="mt-2 h-40 rounded" />}
    </div>
  );
}
```

## Upload Progress

To show how far along an upload is, use `axios` or the `XMLHttpRequest` object. Fetch does not support progress events:

```jsx
function UploadWithProgress() {
  const [progress, setProgress] = useState(0);
  const [uploading, setUploading] = useState(false);

  const handleUpload = (file) => {
    const formData = new FormData();
    formData.append("file", file);
    setUploading(true);

    const xhr = new XMLHttpRequest();
    xhr.upload.onprogress = (e) => {
      if (e.lengthComputable) {
        const percent = Math.round((e.loaded / e.total) * 100);
        setProgress(percent);
      }
    };
    xhr.onload = () => {
      setUploading(false);
      setProgress(0);
    };
    xhr.open("POST", "/api/upload");
    xhr.send(formData);
  };

  return (
    <div>
      {uploading && (
        <div className="w-full bg-gray-200 rounded h-2">
          <div className="bg-green-500 h-2 rounded" style={{ width: `${progress}%` }} />
        </div>
      )}
      <span>{progress}%</span>
    </div>
  );
}
```

## Drag and Drop

Drag and drop feels polished and modern:

```jsx
function DropZone() {
  const [dragging, setDragging] = useState(false);

  const handleDragOver = (e) => {
    e.preventDefault();
    setDragging(true);
  };

  const handleDragLeave = () => setDragging(false);

  const handleDrop = (e) => {
    e.preventDefault();
    setDragging(false);
    const files = Array.from(e.dataTransfer.files);
    uploadFiles(files);
  };

  return (
    <div
      onDragOver={handleDragOver}
      onDragLeave={handleDragLeave}
      onDrop={handleDrop}
      className={`border-2 border-dashed p-8 text-center rounded-lg ${
        dragging ? "border-blue-500 bg-blue-50" : "border-gray-300"
      }`}
    >
      Drag files here or click to browse
    </div>
  );
}
```

## Multiple Files

For multiple files, iterate and upload:

```jsx
const uploadFiles = async (files) => {
  const results = await Promise.all(
    files.map(async (file) => {
      const formData = new FormData();
      formData.append("file", file);
      const res = await fetch("/api/upload", { method: "POST", body: formData });
      return res.json();
    })
  );
  return results;
};
```

Combine these pieces and you have a complete file upload experience: input, preview, progress, drag and drop, and multiple files.
