# FireTube Video Sharing Platform

> ⚠️ This project is not hosted publicly for privacy and security reasons. Unlike typical YouTube clones that rely on YouTube APIs, this implementation includes a complete backend video processing pipeline. ( Don't want to get in trouble :) )

---

## Table of Contents
- [Tech Stack](#tech-stack)
- [Features](#features)
- [High-Level Overview](#high-level-overview)
- [In-Depth Design](#in-depth-design)
- [Challenges and Future Improvements](#challenges-and-future-improvements)

---

## Tech Stack

- **Video Storage:** Google Cloud Storage (for raw and processed videos)
- **Event Handling:** Cloud Pub/Sub (to manage upload events)
- **Video Processing:** Cloud Run + FFmpeg (for video transcoding)
- **Metadata Storage:** Firestore (to store video metadata)
- **API:** Firebase Functions (simple backend API)
- **Web Client:** Next.js (frontend, deployed on Cloud Run)
- **Authentication:** Firebase Auth (user sign-in with Google)

---

## Features

- 🔐 **User Authentication**: Google Sign-In via Firebase Auth  
- 📤 **Video Uploading**: Only authenticated users can upload videos  
- 🎞️ **Video Transcoding**: Converts uploads into multiple resolutions (e.g., 360p, 720p)  
- 📺 **Video Viewing**: Public access to video listings and individual videos  

---

## High-Level Overview

### Architecture Summary

- **Cloud Storage** holds raw and processed videos, offering cost-effective scalability.
- **Cloud Pub/Sub** captures video upload events and queues them for processing.
- **Cloud Run Workers** listen to Pub/Sub events, transcode videos using FFmpeg, and store outputs back in Cloud Storage.
- **Firestore** stores video metadata such as titles and descriptions, enabling video display on the frontend.
- **Firebase Functions** expose basic APIs for video upload and retrieval.
- **Next.js Client** allows authenticated users to upload and view videos.
- **Firebase Auth** manages secure user login with Google.

---

## In-Depth Design

### 1. User Sign-Up

Authentication is managed using **Firebase Auth**, supporting seamless Google Sign-In. Upon login, a user record is created, and additional data (like name and profile image) is stored in **Firestore** via **Firebase Functions**.

### 2. Video Upload

Only logged-in users can upload videos. Uploads are securely handled using signed URLs from **Google Cloud Storage**. The system ties each upload to the user's ID, enabling future enhancements like quotas and user-specific feeds.

### 3. Video Processing

After a video is uploaded:

- A message is pushed to **Cloud Pub/Sub**.
- Workers deployed on **Cloud Run** receive the message and transcode the video using **FFmpeg**.
- Transcoded videos are stored publicly in **Cloud Storage**.
- Metadata (resolution, uploader ID, etc.) is written to **Firestore**.

Benefits of this design:

- Upload and processing pipelines are decoupled.
- Pub/Sub buffers events and enables asynchronous scaling.
- Worker pods scale automatically based on load.

---

## Challenges and Future Improvements

### Known Limitations

1. **Pub/Sub Deadlines**  
   Cloud Run has a 3600s processing limit, but Pub/Sub's max ack deadline is 600s. If processing exceeds this, the message can be stuck. A switch to **Pull Subscriptions** would provide more control over message acknowledgement.

2. **Processing Failures**  
   Failed transcodes can leave messages stuck in the queue. Adding retry logic and resetting the Firestore status to `"undefined"` would improve resilience.

3. **Signed URL Expiry**  
   Uploads must start within 15 minutes. While ongoing uploads continue past expiry, slower networks may struggle.

4. **Streaming Limitations**  
   GCS supports basic streaming but lacks features like **DASH** or **HLS**, which provide adaptive bitrate and chunked delivery.

5. **No CDN Integration**  
   Currently, videos are served directly from GCS. Integrating a **CDN** would reduce latency and improve user experience.

### Future Work

- Display user email and profile photo in the frontend.
- Support multi-video uploads without page refresh.
- Enable custom thumbnails and video descriptions.
- Show uploader details on video pages.
- Add user subscriptions and video channels.
- Implement automatic cleanup of raw video files.
- Integrate a CDN for better global performance.
- Add unit and integration tests for improved maintainability.


