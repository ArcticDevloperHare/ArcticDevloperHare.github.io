# Gif-To-WebP API Conversion Tool Documentation

## Quick guide

The Gif-To-WebP is a *fictional* API which allows you to convert humorous animated GIF and videos memes into :

1.Turn your GIF into an animated WebP files.
2.Convert your GIF into static file of your choice, WebP, jpeg or png(*default*)

You can also choose the frame you want to use as a picture.
To help you with that this API also has an endpoint dedicated to finding the number of frames in a gif animation.

Aside from that this API has three other endpoints meant for

- Downloading the final file.
- Uploading a file.
- Deleting an uploaded file

So, Gif-To-WebP is a beginner friendly API with six total endpoints and a really good documentation to explain it!
**Base URL:** `https://api.giftowebp.com/v1`

---

## Authentication

You need to sign up to use this API ! You must use your API key in header like this: 

```
Authorization: Bearer YOUR_API_KEY
```

To get your API key, [sign up here](https://giftowebp.com/signup).

---

## Endpoints

### 1. Convert GIF to WebP (Animation)

**Endpoint:** `POST /convert`

**Description:** Converts a GIF file (uploaded or from URL) to WebP format with specified compression quality.

#### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `gif_file` | File | Yes* | The GIF file to convert (multipart/form-data upload). Max 50MB. |
| `gif_url` | String | Yes* | URL of the GIF file to convert. |
| `file_id` | String| Yes*| Unique id of a file you have already uploaded.|
| `quality` | Integer | No | Compression quality (0-100). Default: 80. Higher = better quality, larger file. |

*Either `gif_file`, `file_id` OR `gif_url` is required, but not both, `file_id` can only be used if you have uploaded a file.
*Posting both `gif_file` and `gif_url`, or more than one file indentification parameter at a time is not allowed and gives `400 Bad Request Error`.

#### Request Example (File Upload)

```bash
curl -X POST https://api.giftowebp.com/v1/convert/ \
  -H "Content-Type: multipart/form-data" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "gif_file=@animation.gif" 
  -F "quality=85"
```

>NOTE : Maximum file size is 50MB, higher than that gives a 413 Payload Too Large.

#### Request Example (URL)

```bash
curl -X POST https://api.giftowebp.com/v1/convert \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "gif_url": "https://example.com/animation.gif",
    "quality": 85
  }'
```
#### Request Example (Using file_id):

```
curl -X POST https://api.giftowebp.com/v1/convert \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file_id": "file_abc123xyz",
    "quality": 85
  }'
```


#### Response (Success - 200)

```json
{
  "status": "success",
  "file_id" : "webp_abc123xyz",
  "download_url": "https://api.giftowebp.com/files/webp_abc123xyz",
  "file_name": "animation.webp",
  "original_size_kb": 2048,
  "converted_size_kb": 512,
  "compression_ratio": 75,
  "quality": 85,
  "expires_at": "2026-06-10T12:00:00Z"
}
```

**Response Fields:**

- `file_id` (string) Gives the file id to download with GET method endpoint.
- `download_url` (String) — URL to download the converted WebP file
- `file_name` (String) — Name of the generated WebP file
- `original_size_kb` (Integer) — Size of original GIF in kilobytes
- `converted_size_kb` (Integer) — Size of converted WebP in kilobytes
- `compression_ratio` (Integer) — Percentage reduction in file size
- `quality` (Integer) — Quality setting used for conversion
- `expires_at` (String) — When the download link expires

#### Error codes :
> `URL_UNREACHABLE` `RATE_LIMIT_EXCEEDED` `QUALITY_NOT_SUPPORTED` `NOT_ANIMATED_GIF` `NOT_A_GIF` `MISSING_PARAMETER` `INVALID_URL`
> `INVALID_FORMAT` `INVALID_FILE_ID` `INVALID_API_KEY``FILE_TOO_LARGE` `CONVERSION_FAILED` `INVALID_QUALITY` `INVALID_REQUEST`

Check [ Error Code Reference ](#error-code-reference) for more information.


---

### 2. Extract animated Gif to png, jpeg and Webp (Static).

**Description:** "Extract a single frame from your GIF file as a static image"

**Endpoint:**  `POST /extract`

#### Request Parameters 

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
|`gif_file` | File | Yes*    | --  | The GIF file to convert (multipart/form-data upload). Max 50MB. |
|`gif_url` | String | Yes*   | --  | URL of the GIF file to convert. |
|`file_id`|String|  Yes*    |--    | GET output files from an already uploaded file by giving its ID stored in variable `file_id`|
|`end_format` | String | NO  | `png`| You can choose between png, webp, and jpeg, the default is a .png file and you cannot pick quality compression in png. |
|`quality`|Integer| No|80| Compression quality (0-100). Higher = better quality, larger file. Not available for a .png Format|
|`frame`|Integer/String|  No| Mid* | Allows you to choose the frame you want to convert, the variables are : start, end , mid, and your custom frame you enter as an Integer. You can get the total number of frames through endpoint `GET/frames/{file_id}`|


*Either `gif_file`, `file_id` OR `gif_url` is required, but not both, `file_id` can only be used if you have uploaded a file.
*Posting both `gif_file` and `gif_url`, or more than one file identification parameter at a time is not allowed and gives `400 Bad Request Error`.
*The default mid is `n/2` for even numbers of frames and `(n-1)/2` for odd numbers of frames.

#### Example Request :

```
curl -X POST https://api.giftowebp.com/v1/extract \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "gif_url": "https://example.com/animation.gif",
    "end_format": "jpeg",
    "frame": 5,
    "quality": 90
  }'
```

#### Example Response :

```
{
  "status": "success",
  "conversion_id": "conv_jkl789mno",
  "download_url": "https://api.giftowebp.com/files/conv_jkl789mno",
  "file_name": "animation_frame_5.jpeg",
  "output_format": "jpeg",
  "frame_selected": 5,
  "total_frames": 30,
  "original_size_kb": 2048,
  "converted_size_kb": 180,
  "compression_ratio": 91.2,
  "quality": 90,
  "expires_at": "2026-06-30T12:00:00Z"
}
```

**Error Codes :** 
> `INVALID_REQUEST` `MISSING_PARAMETER` `NOT_A_GIF` `FILE_TOO_LARGE` `INVALID_FILE_ID`	`INVALID_FORMAT` `QUALITY_NOT_SUPPORTED` `INVALID_FRAME`
> `NO_FRAMES_FOUND`	`URL_UNREACHABLE`	`INVALID_URL`	`INVALID_API_KEY`	`EXTRACTION_FAILED` `RATE_LIMIT_EXCEEDED` `FRAME_OUT_OF_RANGE`. 


**Example**

```
// FRAME_OUT_OF_RANGE
{
  "status": "error",
  "error_code": "FRAME_OUT_OF_RANGE",
  "message": "Frame index 50 exceeds total frames (30). Use `GET /frames/{file_id}` to check available frames."
}
```

---

### 3. GET Frames of your `gif` file.

***Endpoint:** `GET /frames/{file_id}`

**Description:** Get metadata about frames in a GIF file

>**Parameter Required** : Path variable - file_id.

**Request:** 
```
curl -X GET https://api.giftowebp.com/v1/frames/conv_abc123xyz \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Response:**
```
{
  "status": "success",
  "file_id": "conv_abc123xyz",
  "total_frames": 30,
  "dimensions": {
    "width": 480,
    "height": 270
  },
  "frame_info": {
    "frame_0": {
      "delay": 100,
      "disposal_method": "none"
    },
    "frame_1": {
      "delay": 100,
      "disposal_method": "none"
    }
    // ... continues for all frames
  },
  "duration_ms": 3000
}
```

**Error Codes:**
>`FILE_NOT_FOUND` `NOT_A_GIF` `INVALID_API_KEY` `RATE_LIMIT_EXCEEDED`	

---

### 4. Download Converted File

**Endpoint:** `GET /files/{file_id}`

**Description:** Download a converted file by its ID

>**Parameter Required**: Path variable file_id

**Request:**
```bash
curl -X GET https://api.giftowebp.com/v1/files/conv_abc123xyz \
  -H "Authorization: Bearer YOUR_API_KEY" \
  --output converted_file.webp
```

**Response:** Binary file with appropriate headers
```
HTTP/1.1 200 OK
Content-Type: image/webp
Content-Disposition: attachment; filename="animation.webp"
Content-Length: 524288
```

**Error Codes:**
> `FILE_NOT_FOUND` `INVALID_FILE_ID` `INVALID_API_KEY` `RATE_LIMIT_EXCEEDED`	
---

### 5. Upload a file for Later

**Endpoint**: `POST /upload`

**Purpose:** Upload or provide a URL to a GIF file and return metadata about it without converting.

**Use cases:**
- Get a file_id to use.
- Check if a GIF is valid before converting
- Get total frame count to choose a specific frame
- Preview file size before committing to conversion
- Validate the GIF is animated

---

Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `gif_file` | File | Yes* | The GIF file to upload (multipart/form-data upload). Max 50MB. |
| `gif_url` | String | Yes* | URL of the GIF file to analyze. |

**Note:** *Either `gif_file` OR `gif_url` is required, but not both.

---

#### Example Requests

 **1. Upload File**

```bash
curl -X POST https://api.giftowebp.com/v1/upload \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: multipart/form-data" \
  -F "gif_file=@animation.gif"
```

 **2. Provide URL**

```bash
curl -X POST https://api.giftowebp.com/v1/upload \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "gif_url": "https://example.com/animation.gif"
  }'
```

---

#### Response (Success - 200)

```json
{
  "status": "success",
  "file_id": "file_abc123xyz",
  "file_name": "animation.gif",
  "file_size_kb": 2048,
  "is_animated": true,
  "total_frames": 30,
  "dimensions": {
    "width": 480,
    "height": 270
  },
  "frame_delays_ms": [100, 100, 100, ...],
  "total_duration_ms": 3000,
  "has_transparency": true,
  "color_count": 256,
  "expires_at": "2026-06-30T12:00:00Z"
}
```

---

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `file_id` | String | Unique ID for the uploaded file |
| `file_name` | String | Original filename |
| `file_size_kb` | Integer | File size in kilobytes |
| `is_animated` | Boolean | Whether the GIF is animated |
| `total_frames` | Integer | Number of frames (1 if static) |
| `dimensions` | Object | Width and height of the GIF |
| `frame_delays_ms` | Array | Delay between each frame in milliseconds |
| `total_duration_ms` | Integer | Total animation duration in milliseconds |
| `has_transparency` | Boolean | Whether the GIF contains transparent areas |
| `color_count` | Integer | Number of unique colors in the GIF |
| `expires_at` | String | When the file info expires (ISO 8601) |
---

#### File Retention

- Converted WebP files are stored for **7 days**
- Download URLs expire after 7 days
- Files are automatically deleted after expiration
- You can request immediate deletion through endpoint `DELETE files/{file_id}`

**Error Codes:**
>`INVALID_REQUEST`	`MISSING_PARAMETER`	`NOT_A_GIF` `FILE_TOO_LARGE`	`URL_UNREACHABLE` `INVALID_URL` `INVALID_API_KEY` `RATE_LIMIT_EXCEEDED`.	

---
### 6. DELETE uploaded files.

**Endpoint:** `DELETE /files/{file_id}`

**Purpose:** Delete an uploaded or converted file from the server before it expires.

>**Parameter Required** : Path variable file_id

**Use cases:**
- Clean up files after download
- Save storage/quota by removing unwanted files
- Privacy concerns (remove sensitive content immediately)
- Free up file ID for reuse

**Note:** This endpoint works for both uploaded files (from `/upload`) AND converted files (from `/convert`).

**Required Parameter:** Path variable file_id.


#### Example Request

```bash
curl -X DELETE https://api.giftowebp.com/v1/files/conv_abc123xyz \
  -H "Authorization: Bearer YOUR_API_KEY"
```

#### Response (Success - 200)

```json
{
  "status": "success",
  "message": "File deleted successfully",
  "file_id": "conv_abc123xyz",
  "deleted_at": "2026-06-23T14:30:00Z"
}
```
---

**Error Codes :**
> `FILE_NOT_FOUND` `FILE_ALREADY_DELETED` `INVALID_FILE_ID`	`INVALID_API_KEY` `RATE_LIMIT_EXCEEDED`	

---


## Error Code Reference 

This API returns descriptive error messages. Here are the errors you may encounter:

### Master Table for Reference:


| Error Code | HTTP Status | Meaning | Endpoints |
|-----------|------------|---------|-----------|
| `CONVERSION_FAILED` | 500 | Server-side conversion error | `/convert` |
| `EXTRACTION_FAILED` | 500 | Server-side extraction error | `/extract` |
| `FILE_ALREADY_DELETED` | 410 | File was already deleted | `DELETE /files` |
| `FILE_NOT_FOUND` | 404 | File doesn't exist or expired | `GET /files`, `GET /frames`, `DELETE /files` |
| `FILE_TOO_LARGE` | 413 | File exceeds 50MB limit | `/upload`, `/convert`, `/extract` |
| `FRAME_OUT_OF_RANGE` | 400 | Frame index exceeds total frames | `/convert`, `/extract` |
| `INVALID_API_KEY` | 401 | API key missing or invalid | All endpoints |
| `INVALID_FILE_ID` | 400 | File ID format is invalid | `GET /files`, `GET /frames`, `DELETE /files`, `/convert`, `/extract` |
| `INVALID_FORMAT` | 400 | Invalid output format specified | `/convert`, `/extract` |
| `INVALID_FRAME` | 400 | Invalid frame parameter | `/convert`, `/extract` |
| `INVALID_QUALITY` | 400 | Quality must be 0-100 | `/convert`, `/extract` |
| `INVALID_REQUEST` | 400 | Missing or conflicting parameters | `/upload`, `/convert`, `/extract` |
| `INVALID_URL` | 400 | URL doesn't point to GIF | `/upload`, `/convert`, `/extract` |
| `MISSING_PARAMETER` | 400 | Required parameter missing | `/upload`, `/convert`, `/extract` |
| `NO_FRAMES_FOUND` | 400 | GIF has no frames | `/extract` |
| `NOT_A_GIF` | 400 | File is not a valid GIF | `/upload`, `/convert`, `/extract`, `GET /frames` |
| `NOT_ANIMATED_GIF` | 400 | GIF is static (not animated) | `/convert` |
| `QUALITY_NOT_SUPPORTED` | 400 | Quality not supported for PNG | `/convert`, `/extract` |
| `RATE_LIMIT_EXCEEDED` | 429 | Daily conversion limit exceeded | All endpoints |
| `URL_UNREACHABLE` | 400 | Cannot access provided URL | `/upload`, `/convert`, `/extract` |


---

### Examples 

#### 400 Bad Request

```json
{
  "status": "error",
  "error_code": "INVALID_REQUEST",
  "message": "Either gif_file or gif_url must be provided, but not both"
}
```

#### 400 Bad Request — Invalid File Type

```json
{
  "status": "error",
  "error_code": "NOT_A_GIF",
  "message": "The file provided is not a valid GIF file. Please check the file format."
}
```

#### 400 Bad Request — Animated GIF Required

```json
{
  "status": "error",
  "error_code": "NOT_ANIMATED_GIF",
  "message": "The GIF file provided is not animated. This API only converts animated GIFs."
}
```

#### 413 Payload Too Large

```json
{
  "status": "error",
  "error_code": "FILE_TOO_LARGE",
  "message": "File size exceeds 50MB limit. Your file is 75MB."
}
```

#### 401 Unauthorized

```json
{
  "status": "error",
  "error_code": "INVALID_API_KEY",
  "message": "The API key provided is invalid or has expired."
}
```

#### 500 Server Error

```json
{
  "status": "error",
  "error_code": "CONVERSION_FAILED",
  "message": "An error occurred during conversion. Please try again later."
}
```

---

## Quality Settings

Choose compression quality based on your needs:

- **90-100:** Highest quality, minimal compression. Use for professional work where quality is critical.
- **70-89:** Balanced quality and file size. Recommended for most use cases.
- **50-69:** Lower quality, significant compression. Use when file size is critical.
- **Below 50:** Very low quality. Not recommended.

**Example:**
```bash
curl -X POST https://api.giftowebp.com/v1/convert \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "gif_file=@large_animation.gif" \
  -F "quality=75"
```

---

## Rate Limits

- **Free tier:** 100 conversions per hour
- **Pro tier:** 10,000 conversions per hour
- **Enterprise:** Unlimited

Rate limit info is included in response headers:
```
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 7
X-RateLimit-Reset: 1717944000
```

---

## Best Practices

1. **Always specify quality:** Don't rely on defaults; test quality levels for your use case
2. **Handle errors gracefully:** Check error codes and show user-friendly messages
3. **Download files promptly:** Don't wait until expiration; download immediately after conversion
4. **Use URLs when possible:** Uploading files directly is slower; use `gif_url` when you have a public URL
5. **Test with small files first:** Before processing large files, test your integration with small GIFs

---

## Example Integration (Python)

```python
import requests

def convert_gif_to_webp(gif_file_path, quality=80):
    url = "https://api.giftowebp.com/v1/convert"
    headers = {
        "Authorization": "Bearer YOUR_API_KEY"
    }
    
    files = {
        "gif_file": open(gif_file_path, "rb")
    }
    
    data = {
        "quality": quality
    }
    
    response = requests.post(url, headers=headers, files=files, data=data)
    
    if response.status_code == 200:
        result = response.json()
        print(f"Success! Download: {result['download_url']}")
        return result
    else:
        error = response.json()
        print(f"Error: {error['message']}")
        return None

# Usage
convert_gif_to_webp("animation.gif", quality=85)
```

---

## Support

- **Email:** support@giftowebp.com
- **Documentation:** https://docs.giftowebp.com
- **Status Page:** https://status.giftowebp.com
