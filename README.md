# Employee Management

## Overview

Employee Management is a full‑stack application designed to manage employee data efficiently. It provides a backend API, a frontend user interface, and a clear API specification to support CRUD operations and future extensibility.

This repository is structured to separate concerns between backend services, frontend presentation, and documentation.

## Project Structure

```
.
├── backend                  # Backend application (API)
├── frontend                 # Frontend application (UI)
├── openapi.yaml             # OpenAPI / Swagger specification
├── ARCHITECTURE.md          # System architecture documentation
├── Jenkinsfile              # CI/CD pipeline configuration
├── .gitignore
└── LICENSE
```

## Features

* Create, read, update, and delete employee records
* RESTful API architecture
* Frontend interface for managing employees
* API documentation using OpenAPI
* CI/CD pipeline integration

> Adjust this list if additional features are implemented.

## Prerequisites

Ensure the following tools are installed before running the project:

* Git
* Node.js (v14 or later)
* npm or Yarn
* A database system if required by the backend

## Installation

Clone the repository:

```bash
git clone https://github.com/chaimaeddib2005/Employee_Management.git
cd Employee_Management
```

### Backend Setup

```bash
cd backend
npm install
```

### Frontend Setup

```bash
cd ../frontend
npm install
```

## Running the Application

### Backend

From the `backend` directory:

```bash
npm start
```

or

```bash
npm run dev
```

### Frontend

From the `frontend` directory:

```bash
npm start
```

or

```bash
npm run dev
```

The frontend will be available on the configured local development port.

## API Documentation

The API specification is defined in the `openapi.yaml` file. It can be viewed using Swagger UI or any OpenAPI‑compatible tool.

Example:

```bash
npx swagger-ui-dist openapi.yaml
```

## Environment Variables

Example backend `.env` configuration:

```
PORT=4000
DATABASE_URL=your_database_url
JWT_SECRET=your_secret_key
```

Example frontend `.env` configuration:

```
VITE_API_BASE_URL=http://localhost:4000/api
```

Adjust variables based on your setup.

## Usage

Once the application is running, users can:

* View employee records
* Add new employees
* Update existing employee information
* Delete employees

## Contributing

Contributions are welcome.

1. Fork the repository
2. Create a new branch
3. Commit your changes
4. Open a pull request

Follow clean code practices and write meaningful commit messages.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.
