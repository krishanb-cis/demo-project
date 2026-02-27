# CRUD Application - Node.js & Vue.js

A full-stack CRUD (Create, Read, Update, Delete) application with Node.js Express backend, Vue.js 3 frontend, and MongoDB database.

## 📋 Features

- ✅ **Create** - Add new items with title, description, and status
- ✅ **Read** - View all items in a beautiful grid layout
- ✅ **Update** - Edit existing items
- ✅ **Delete** - Remove items with confirmation
- 🎨 **Responsive Design** - Works on desktop, tablet, and mobile
- 🔄 **Real-time Updates** - Instant UI updates on CRUD operations

## 🗂️ Project Structure

```
demo-project/
├── backend/                    # Node.js Express API
│   ├── models/
│   │   └── Item.js            # MongoDB schema
│   ├── controllers/
│   │   └── itemController.js  # Business logic
│   ├── routes/
│   │   └── itemRoutes.js      # API routes
│   ├── server.js              # Main server file
│   ├── .env                   # Environment variables
│   └── package.json           # Backend dependencies
│
├── frontend/                   # Vue.js 3 Frontend
│   ├── src/
│   │   ├── App.vue           # Main component
│   │   ├── main.js           # Vue app entry
│   │   └── components/       # Reusable components
│   ├── public/               # Static assets
│   ├── index.html            # HTML entry point
│   ├── vite.config.js        # Vite configuration
│   └── package.json          # Frontend dependencies
│
├── README.md                  # Main documentation
└── .github/
    └── copilot-instructions.md
```

## 🚀 Getting Started

### Prerequisites

- **Node.js** (v14 or higher)
- **MongoDB** (running locally on port 27017)
- **npm** or **yarn**

### Backend Setup

1. Navigate to the backend directory:
   ```bash
   cd backend
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create a `.env` file (already in repo) and ensure a JWT secret is set:
   ```env
   MONGODB_URI=mongodb://localhost:27017/crud_app
   PORT=5000
   JWT_SECRET=your_jwt_secret_here
   ```

4. Start the backend server:
   ```bash
   npm start
   # or for development with auto-reload:
   npm run dev
   ```

3. Ensure MongoDB is running:
   ```bash
   # On Windows (if MongoDB is installed)
   mongod
   ```

4. Start the backend server:
   ```bash
   npm start
   # or for development with auto-reload:
   npm run dev
   ```

   The backend will run on `http://localhost:5000`

### Frontend Setup

1. Open a new terminal and navigate to the frontend directory:
   ```bash
   cd frontend
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Start the development server:
   ```bash
   npm run dev
   ```

   The frontend will run on `http://localhost:3000`

## 📡 API Endpoints

### Items

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register new user (returns token) |
| POST | `/api/auth/login` | Authenticate user (returns token) |
| GET | `/api/items` | Get all items (requires Authorization header) |
| GET | `/api/items/:id` | Get single item by ID (requires Authorization) |
| POST | `/api/items` | Create new item (requires Authorization) |
| PUT | `/api/items/:id` | Update item by ID (requires Authorization) |
| DELETE | `/api/items/:id` | Delete item by ID (requires Authorization) |

### Request/Response Format

**Create/Update Item (POST/PUT):**
```json
{
  "title": "Task Title",
  "description": "Task Description",
  "status": "pending" // or "completed"
}
```

**Response:**
```json
{
  "_id": "ObjectId",
  "title": "Task Title",
  "description": "Task Description",
  "status": "pending",
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-15T10:30:00Z"
}
```

## 🎯 Usage

1. **Access the Application:**
   - Open browser and go to `http://localhost:3000`

2. **Authentication Flow:**
   - Register a new account or login with existing credentials.
   - After successful login you will be redirected to the CRUD interface.

## 🧪 Testing

You can manually verify everything works by performing the following steps:

1. Start backend and frontend as described above.
2. In the browser, register a user at the login form.
3. Use the login form to authenticate; check that a token is stored (see `localStorage`).
4. Create, edit, delete items; ensure operations succeed and persist in MongoDB.
5. Open Developer Tools → Network to confirm `Authorization: Bearer <token>` header on requests.
6. Try accessing `/api/items` with a missing or invalid token via `curl` or Postman; the server should return 401.

Automated tests are not included but could be added later under `backend/tests` or with a front‑end testing framework.

2. **Create Items:**
   - Fill in the form with title and description
   - Select status (pending/completed)
   - Click "Add Item"

3. **View Items:**
   - All items appear in a grid layout below the form
   - Shows creation date, status badge, and action buttons

4. **Edit Items:**
   - Click "Edit" button on any item
   - Form will populate with item data
   - Make changes and click "Update Item"

5. **Delete Items:**
   - Click "Delete" button on any item
   - Confirm deletion in the popup

## 🛠️ Tech Stack

### Backend
- **Express.js** - Web framework
- **Mongoose** - MongoDB object modeling
- **CORS** - Cross-Origin Resource Sharing
- **Nodemon** - Auto-reload during development

### Frontend
- **Vue.js 3** - Progressive JavaScript framework
- **Vite** - Build tool and dev server
- **Axios** - HTTP client

### Database
- **MongoDB** - NoSQL database

## 🔧 Configuration

### Backend (.env)
```
MONGODB_URI=mongodb://localhost:27017/crud_app
PORT=5000
```

### Frontend (vite.config.js)
- Dev server port: 3000
- Proxy API calls to backend: /api -> http://localhost:5000

## 📝 Development

### Adding New Features

1. **Add Backend Endpoint:**
   - Create controller method in `backend/controllers/itemController.js`
   - Add route in `backend/routes/itemRoutes.js`

2. **Update Frontend:**
   - Modify `frontend/src/App.vue` component
   - Update API calls with new endpoints

3. **Update Database Schema:**
   - Modify `backend/models/Item.js` if changing data structure

## 🐛 Troubleshooting

### MongoDB Connection Error
- Ensure MongoDB is running: `mongod`
- Check `MONGODB_URI` in `.env` file

### CORS Error
- Backend CORS is enabled for all origins
- Check that both servers are running

### Port Already in Use
- Backend: Change PORT in `.env`
- Frontend: Change port in `vite.config.js`

### API Not Responding
- Verify backend is running on port 5000
- Check network tab in browser DevTools
- Review console for error messages

## 📦 Building for Production

### Frontend Build
```bash
cd frontend
npm run build
# Creates dist/ folder with optimized build
```

### Process
1. Build frontend: `npm run build`
2. Serve frontend with a production server
3. Deploy backend to a server
4. Update API URL in frontend for production

## 🤝 Contributing

Feel free to fork, modify, and improve this project!

## 📄 License

ISC License

## 📧 Support

For issues or questions, refer to the error messages in browser console and terminal output.

