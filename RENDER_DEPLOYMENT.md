# Render Deployment Guide

This guide outlines the steps to deploy the Momentz application (Backend and Frontend) to [Render](https://render.com).

## 1. Backend Deployment (Web Service)

### Setup Steps
1. Create a new **Web Service** on Render.
2. Connect your GitHub repository.
3. Set the **Root Directory** to `backend`.

### Build & Start Commands
- **Runtime**: Node
- **Build Command**: `npm install && npx prisma generate && npx prisma migrate deploy`
- **Start Command**: `npm start`

### Environment Variables
| Variable | Description |
| :--- | :--- |
| `DATABASE_URL` | Your PostgreSQL connection string (Neon/Render DB) |
| `JWT_SECRET` | A strong, random string for authentication |
| `CLOUDINARY_CLOUD_NAME` | Your Cloudinary account name |
| `CLOUDINARY_API_KEY` | Your Cloudinary API key |
| `CLOUDINARY_API_SECRET` | Your Cloudinary API secret |
| `FRONTEND_URL` | The URL of your deployed frontend |
| `NODE_ENV` | Set to `production` |

---

## 2. Frontend Deployment (Static Site)

### Setup Steps
1. Create a new **Static Site** on Render.
2. Connect your GitHub repository.
3. Set the **Root Directory** to `frontend`.

### Build & Start Commands
- **Build Command**: `npm install && npm run build`
- **Publish Directory**: `dist`

### Environment Variables
| Variable | Description |
| :--- | :--- |
| `VITE_API_URL` | The URL of your deployed backend |
| `VITE_CLOUDINARY_BASE_URL` | Your Cloudinary base URL |

> [!IMPORTANT]
> For React Router support, go to **Redirects/Rewrites** in Render settings and add:
> - **Source**: `/*`
> - **Destination**: `/index.html`
> - **Action**: `Rewrite`

---

## 3. Database
If you didn't create a DB yet, you can create a **PostgreSQL** instance on Render and use its **Internal Database URL** as the `DATABASE_URL` for the backend.
