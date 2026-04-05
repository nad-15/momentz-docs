# Vercel Deployment Audit for Momentz

This document outlines the major architectural "Red Flags" that will cause the current Node/Express backend to crash or throw errors if deployed to Vercel as-is. Vercel is built for **Serverless Functions**, which behave very differently from a traditional long-running Node server.

## 🚨 1. The "413 Payload Too Large" Crash (Media Uploads)

### The Problem
Currently, when a user creates a Moment, the React frontend sends the video/image to the Node backend (`/upload`), which holds it in RAM using `multer` and then sends it to Cloudinary. 
* **Vercel has a strict 4.5 MB limit** on the body of all requests to its servers (Serverless Functions). 
* Since Momentz allows videos up to 30MB, any moment larger than 4.5MB will instantly fail with a `413 Payload Too Large` error. 
* Furthermore, Vercel has a **10-second request timeout** on the free tier. Slower uploads will simply drop dead mid-upload.

### The Fix: Direct Cloudinary Uploads
You need to refactor the upload flow entirely:
1. The React frontend asks the Node backend for a "secure signature".
2. The React frontend uploads the 30MB video **directly** to Cloudinary using that signature, bypassing Vercel completely.
3. The frontend then sends just the resulting `mediaUrl` to the backend to save in the database.

## 🚨 2. Database Connection Exhaustion

### The Problem
Serverless functions are stateless. Every single time a user hits an endpoint, Vercel spins up a new instance of your backend function. Each instance opens a brand new, separate connection to your PostgreSQL database.
If 50 people visit the app at once, Vercel opens 50 connections. This will quickly exhaust your database connection limits and crash the DB with a `too many clients` error.

### The Fix: Connection Pooling
Ensure your `DATABASE_URL` connects through a **Connection Pooler**. If you are using Supabase, you must use the port `6543` connection string (the pooler) rather than the default `5432` direct connection string. 
*(Note: You are already using `@prisma/adapter-pg` which is great, but the connection string must still point to a pooler).*

## 🚨 3. Configuration & Routing (`vercel.json`)

### The Problem
Vercel's build system doesn't automatically know how to run a monolithic Express app (`server.js`). If you just push this repository, the backend will completely fail to build or route requests correctly.

### The Fix: `vercel.json` Setup
You need to create a `vercel.json` file in your root directory to tell Vercel to route all traffic (e.g., `/api/*`) into your `server.js` file properly, treating it as a single serverless function.

---

## 💡 Recommendation

Deploying the **Frontend (React/Vite)** to Vercel is a perfect fit.

However, deploying a monolithic Express backend that handles large video uploads to Vercel is famously painful due to the limits mentioned above.

**Alternative:** 
Deploy the backend to a non-serverless platform like **Render** or **Railway**. 
* These platforms run traditional, long-running Node servers (just like your local machine running `npm run dev`).
* You won't have to change your media upload code at all.
* You won't have to worry about Serverless function timeouts (10s limit).
* You can still host the Frontend on Vercel for maximum speed and global CDN delivery.
