Creating a **PEAN (PostgreSQL, Express.js, Angular, Node.js)** Portfolio application involves setting up a full-stack application with PostgreSQL as the database, Express.js and Node.js for the backend, and Angular for the frontend. We'll use Docker Compose to manage PostgreSQL and pgAdmin, and include startup scripts for database schema creation and seed data. Additionally, we'll provide a `.devcontainer` configuration for development in Visual Studio Code.

---

### **Step 1: Project Structure**
The project will be organized into three main folders:
```
pean-portfolio/
├── backend/
├── frontend/
├── db/
│   ├── docker-compose.yml
│   ├── schema.sql
│   ├── seed.sql
│   └── init.sh
└── .devcontainer/
    ├── devcontainer.json
    └── Dockerfile
```

---

### **Step 2: Set Up PostgreSQL and pgAdmin with Docker Compose**

1. **Create the `db` Folder**:
   - Inside the `db` folder, create the following files:
     - `docker-compose.yml`
     - `schema.sql` (for database schema)
     - `seed.sql` (for seed data)
     - `init.sh` (startup script to initialize the database)

2. **`docker-compose.yml`**:
   - Define PostgreSQL and pgAdmin services:
     ```yaml
     version: '3.8'

     services:
       db:
         image: postgres:latest
         container_name: postgres_db
         environment:
           POSTGRES_USER: postgres
           POSTGRES_PASSWORD: postgres
           POSTGRES_DB: portfolio_db
         ports:
           - "5432:5432"
         volumes:
           - postgres_data:/var/lib/postgresql/data
           - ./schema.sql:/docker-entrypoint-initdb.d/schema.sql
           - ./seed.sql:/docker-entrypoint-initdb.d/seed.sql

       pgadmin:
         image: dpage/pgadmin4:latest
         container_name: pgadmin
         environment:
           PGADMIN_DEFAULT_EMAIL: admin@example.com
           PGADMIN_DEFAULT_PASSWORD: admin
         ports:
           - "5050:80"
         depends_on:
           - db

     volumes:
       postgres_data:
     ```

3. **`schema.sql`**:
   - Define the database schema:
     ```sql
     CREATE TABLE portfolio_items (
       id SERIAL PRIMARY KEY,
       title VARCHAR(255) NOT NULL,
       description TEXT,
       image_url VARCHAR(255)
     );
     ```

4. **`seed.sql`**:
   - Add seed data:
     ```sql
     INSERT INTO portfolio_items (title, description, image_url) VALUES
     ('Project 1', 'This is the first project', 'https://example.com/project1.jpg'),
     ('Project 2', 'This is the second project', 'https://example.com/project2.jpg');
     ```

5. **`init.sh`**:
   - Create a startup script to initialize the database:
     ```bash
     #!/bin/bash
     docker-compose up -d
     echo "PostgreSQL and pgAdmin are running."
     echo "Access pgAdmin at http://localhost:5050"
     ```

6. **Run the Database**:
   - Make the `init.sh` script executable:
     ```bash
     chmod +x db/init.sh
     ```
   - Start the database:
     ```bash
     cd db
     ./init.sh
     ```

---

### **Step 3: Set Up the Backend (Node.js + Express.js)**

1. **Create the `backend` Folder**:
   - Inside the `backend` folder, initialize a Node.js project:
     ```bash
     npm init -y
     ```

2. **Install Dependencies**:
   - Install required packages:
     ```bash
     npm install express pg sequelize cors dotenv
     ```

3. **Set Up Sequelize**:
   - Initialize Sequelize:
     ```bash
     npx sequelize-cli init
     ```
   - Update `config/config.json` for PostgreSQL:
     ```json
     {
       "development": {
         "username": "postgres",
         "password": "postgres",
         "database": "portfolio_db",
         "host": "db",
         "dialect": "postgres"
       }
     }
     ```

4. **Create a Model**:
   - Generate a model for portfolio items:
     ```bash
     npx sequelize-cli model:generate --name PortfolioItem --attributes title:string,description:text,imageUrl:string
     ```

5. **Run Migrations**:
   - Apply the migrations:
     ```bash
     npx sequelize-cli db:migrate
     ```

6. **Create API Endpoints**:
   - In `backend/routes/portfolio.js`:
     ```javascript
     const express = require('express');
     const router = express.Router();
     const { PortfolioItem } = require('../models');

     // Get all portfolio items
     router.get('/', async (req, res) => {
       try {
         const portfolioItems = await PortfolioItem.findAll();
         res.json(portfolioItems);
       } catch (err) {
         res.status(500).json({ message: err.message });
       }
     });

     module.exports = router;
     ```

7. **Update `backend/server.js`**:
   - Set up the Express server:
     ```javascript
     const express = require('express');
     const cors = require('cors');
     const dotenv = require('dotenv');
     const portfolioRouter = require('./routes/portfolio');

     dotenv.config();

     const app = express();
     const PORT = process.env.PORT || 5000;

     app.use(cors());
     app.use(express.json());

     app.use('/api/portfolio', portfolioRouter);

     app.listen(PORT, () => {
       console.log(`Server running on port ${PORT}`);
     });
     ```

---

### **Step 4: Set Up the Frontend (Angular)**

1. **Create the `frontend` Folder**:
   - Generate a new Angular application:
     ```bash
     ng new frontend
     ```

2. **Create Components**:
   - Generate components for the portfolio:
     ```bash
     cd frontend
     ng generate component home
     ng generate component about
     ng generate component projects
     ng generate component contact
     ```

3. **Set Up Routing**:
   - Update `src/app/app-routing.module.ts`:
     ```typescript
     const routes: Routes = [
       { path: '', component: HomeComponent },
       { path: 'about', component: AboutComponent },
       { path: 'projects', component: ProjectsComponent },
       { path: 'contact', component: ContactComponent },
     ];
     ```

4. **Fetch Data from the Backend**:
   - In `src/app/services/portfolio.service.ts`:
     ```typescript
     import { Injectable } from '@angular/core';
     import { HttpClient } from '@angular/common/http';

     @Injectable({
       providedIn: 'root',
     })
     export class PortfolioService {
       private apiUrl = 'http://localhost:5000/api/portfolio';

       constructor(private http: HttpClient) {}

       getPortfolioItems() {
         return this.http.get(this.apiUrl);
       }
     }
     ```

5. **Display Data**:
   - In `src/app/projects/projects.component.ts`:
     ```typescript
     import { Component, OnInit } from '@angular/core';
     import { PortfolioService } from '../services/portfolio.service';

     @Component({
       selector: 'app-projects',
       templateUrl: './projects.component.html',
       styleUrls: ['./projects.component.css'],
     })
     export class ProjectsComponent implements OnInit {
       portfolioItems: any;

       constructor(private portfolioService: PortfolioService) {}

       ngOnInit(): void {
         this.portfolioService.getPortfolioItems().subscribe((data) => {
           this.portfolioItems = data;
         });
       }
     }
     ```

---

### **Step 5: Set Up `.devcontainer` for Development**

1. **Create `.devcontainer` Folder**:
   - Add `devcontainer.json` and `Dockerfile`.

2. **`devcontainer.json`**:
   ```json
   {
     "name": "PEAN Portfolio",
     "dockerComposeFile": "../db/docker-compose.yml",
     "service": "backend",
     "workspaceFolder": "/workspace",
     "shutdownAction": "stopCompose",
     "extensions": ["dbaeumer.vscode-eslint", "angular.ng-template"]
   }
   ```

3. **`Dockerfile`**:
   ```dockerfile
   FROM node:18
   WORKDIR /workspace
   COPY . .
   RUN npm install -g @angular/cli
   ```

---

### **Step 6: Start the Application**

1. **Start the Database**:
   ```bash
   cd db
   ./init.sh
   ```

2. **Start the Backend**:
   ```bash
   cd ../backend
   npm start
   ```

3. **Start the Frontend**:
   ```bash
   cd ../frontend
   ng serve
   ```

---

### **Step 7: Access the Application**

- Frontend: `http://localhost:4200`
- Backend: `http://localhost:5000`
- pgAdmin: `http://localhost:5050`

---

This setup provides a fully functional PEAN stack application with PostgreSQL, Docker Compose, and a `.devcontainer` configuration for seamless development.
