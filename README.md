# Thoth

<p align="center">
  <strong>A Multitier Domain-Specific Language for Full-Stack Web Development</strong>
</p>

> **⚠️ DISCLAIMER**
> 
> This is a proof-of-concept academic project created as part of a master's dissertation. It is not intended for production use and is currently unmaintained. Use at your own risk.

## Table of Contents

- [Overview](#overview)
- [What Makes Thoth Different](#what-makes-thoth-different)
- [Key Features](#key-features)
- [Quick Example](#quick-example)
- [How It Works](#how-it-works)
- [Language Features](#language-features)
- [Getting Started](#getting-started)
- [Example Applications](#example-applications)
- [Project Structure](#project-structure)
- [Learn More](#learn-more)
- [Acknowledgments](#acknowledgments)

## Overview

**Thoth** is a statically-typed domain-specific language (DSL) that enables developers to build complete web applications—database schema, server API, and client UI—in a single, unified language. Write once in Thoth, and the compiler generates clean, human-readable TypeScript code for both your Node.js backend and React frontend.

### The Problem

Traditional full-stack web development requires:
- Multiple programming languages (SQL, TypeScript/JavaScript, potentially others)
- Separate codebases for database migrations, backend APIs, and frontend UIs
- Manual synchronization of data models across tiers
- Extensive boilerplate code for CRUD operations, authentication, and real-time features
- Context switching between different paradigms and tools

### The Solution

Thoth unifies the entire stack into a single declarative language where you define:
- **Data models** (compiled to PostgreSQL schema via Prisma)
- **Queries** (compiled to Express.js API routes with type-safe validation)
- **Components** (compiled to React components with TypeScript)
- **Pages** (compiled to React pages with routing)
- **Authentication & Authorization** (automatically generated and integrated)

## What Makes Thoth Different

### 1. **Truly Multitier**
Unlike frameworks that simply share code between tiers, Thoth lets you declare your application's behavior at a higher level of abstraction. The compiler understands the relationships between database models, API endpoints, and UI components, generating all the glue code automatically.

### 2. **Human-Readable Output**
Thoth compiles to clean, idiomatic TypeScript code that you can read, understand, modify, and extend. No black-box magic—just well-structured React components, Express routes, and Prisma schemas.

### 3. **Real-Time by Default**
Every query automatically supports real-time updates via Server-Sent Events (SSE). Your UI components subscribe to data changes and update automatically—no WebSocket configuration needed.

### 4. **Framework-Agnostic Design**
While the default compiler targets Node.js + Express + Prisma + PostgreSQL on the backend and React + Vite on the frontend, the language itself is not tied to these technologies. The architecture supports alternative compilation targets.

### 5. **Type Safety Across Tiers**
Changes to your data models automatically propagate through your API contracts and UI components, maintaining type safety across the entire stack.

## Key Features

✅ **Declarative Data Modeling** - Define models with fields, relationships, and attributes  
✅ **Automatic CRUD Generation** - Create, Read, Update, Delete operations with minimal code  
✅ **Built-in Authentication** - User signup, login, logout with session management  
✅ **Fine-Grained Authorization** - Route-level and query-level permissions  
✅ **Real-Time Synchronization** - SSE-based live updates across all connected clients  
✅ **Component System** - Declare forms, buttons, and custom components with styling  
✅ **JSX-like Syntax** - Familiar component rendering with template expressions  
✅ **TypeScript Integration** - Leverage the entire npm ecosystem  
✅ **Custom Components** - Write raw React/TypeScript when needed  
✅ **Client Dependencies** - Specify npm packages to include in your project  

## Quick Example

Here's a simple todo application in Thoth:

```thoth
app Todo {
  title: "My Todo App",
  auth: {
    userModel: User,
    idField: id,
    usernameField: username,
    passwordField: password,
    onSuccessRedirectTo: "/",
    onFailRedirectTo: "/login"
  }
}

model User {
  id       Int      @id
  username String   @unique
  password String
  tasks    Task[]
}

model Task {
  id     Int     @id
  title  String
  isDone Boolean @default(false)
  user   User    @relation(userId, id)
  userId Int
}

@model(Task)
@permissions(IsAuth, OwnsRecord)
query<FindMany> getTasks {
  search: [title, isDone]
}

@model(Task)
@permissions(IsAuth)
query<Create> createTask {
  data: {
    fields: [title],
    relationFields: {
      user: connect id with userId
    }
  }
}

component<Create> TaskForm {
  actionQuery: createTask(),
  formInputs: {
    title: {
      input: {
        type: TextInput,
        placeholder: "New task...",
        isVisible: true
      }
    },
    user: {
      input: {
        type: RelationInput,
        isVisible: false,
        defaultValue: connect id with LoggedInUser.id
      }
    }
  },
  formButton: {
    name: "Add Task",
    style: "bg-blue-500 text-white px-4 py-2 rounded"
  }
}

@route("/")
@permissions(IsAuth)
page Home {
  render(
    <div>
      <h1>{"My Tasks"}</h1>
      <TaskForm />
      <TasksComponent />
    </div>
  )
}
```

This compiles to a complete full-stack application with PostgreSQL database, Express REST API, and React frontend—all with real-time synchronization.

## How It Works

```
┌─────────────────┐
│  .thoth file    │  ← Your application code
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Thoth Compiler │  ← OCaml-based compiler (lexer, parser, type checker, code generator)
└────────┬────────┘
         │
         ├──────────────────┬──────────────────┐
         ▼                  ▼                  ▼
┌────────────────┐  ┌──────────────┐  ┌─────────────────┐
│  TypeScript    │  │  TypeScript  │  │  Prisma Schema  │
│  React Client  │  │  Express API │  │  + Migrations   │
│  (Vite)        │  │  (Node.js)   │  │  (PostgreSQL)   │
└────────────────┘  └──────────────┘  └─────────────────┘
```

**Compilation Pipeline:**

1. **Lexical Analysis** - Tokenizes the `.thoth` source file
2. **Parsing** - Builds an Abstract Syntax Tree (AST)
3. **Type Checking** - Validates models, queries, components, and pages
4. **Specification Generation** - Creates intermediate representations for each tier
5. **Code Generation** - Uses Jinja2 templates to generate TypeScript and Prisma code
6. **Output** - Produces two directories: `server/` and `client/` with complete applications

## Language Features

### Models

Define your data schema with Prisma-inspired syntax:

```thoth
model User {
  id         Int       @id
  email      String    @unique
  username   String
  posts      Post[]
  createdAt  DateTime  @default(Now)
}

model Post {
  id        Int      @id
  title     String
  content   String
  author    User     @relation(authorId, id)
  authorId  Int
  published Boolean  @default(false)
}
```

### Queries

Declare type-safe database operations:

```thoth
@model(Post)
@permissions(IsAuth)
query<FindMany> getPosts {
  search: [title, content, published]
}

@model(Post)
@permissions(IsAuth, OwnsRecord)
query<Update> updatePost {
  where: id,
  data: {
    fields: [title, content, published]
  }
}

@model(Post)
@permissions(IsAuth, OwnsRecord)
query<Delete> deletePost {
  where: id
}
```

**Supported query types:** `FindUnique`, `FindMany`, `Create`, `Update`, `Delete`

### Components

Build UI components declaratively:

**Action Forms:**
```thoth
component<Create> PostForm {
  actionQuery: createPost(),
  formInputs: {
    title: {
      label: { name: "Title" },
      input: {
        type: TextInput,
        placeholder: "Enter title"
      }
    },
    content: {
      label: { name: "Content" },
      input: {
        type: TextInput,
        placeholder: "Write your post..."
      }
    }
  },
  formButton: {
    name: "Publish",
    style: "bg-green-500 text-white px-4 py-2"
  }
}
```

**Data Display Components:**
```thoth
component<FindMany> PostsList {
  findQuery: getPosts() as posts,
  onLoading: render(<div>{"Loading..."}</div>),
  onError: render(<div>{"Error loading posts"}</div>),
  onSuccess: render(
    <>
      [% for post in posts %]
        <PostCard post={post} />
      [% endfor %]
    </>
  )
}
```

**Custom Components:**
```thoth
component<Custom> PostCard(post: Post) {
  imports: [|
    import { useState } from "react";
    import { Post } from "@/types";
  |],
  fn: [|
    const [expanded, setExpanded] = useState(false);
    
    return (
      <div className="border rounded p-4">
        <h3>{post.title}</h3>
        <button onClick={() => setExpanded(!expanded)}>
          {expanded ? "Show less" : "Show more"}
        </button>
        {expanded && <p>{post.content}</p>}
      </div>
    );
  |]
}
```

### Pages

Define routes and page components:

```thoth
@route("/")
@permissions(IsAuth)
page Home {
  render(
    <div>
      <h1>{"Home Page"}</h1>
      <PostsList />
    </div>
  )
}

@route("/login")
page Login {
  render(
    <div>
      <h1>{"Login"}</h1>
      <LoginForm />
    </div>
  )
}
```

### Authentication

Built-in authentication configuration:

```thoth
app MyApp {
  title: "My Application",
  auth: {
    userModel: User,
    idField: id,
    isOnlineField: isOnline,      // optional
    lastActiveField: lastActive,  // optional
    usernameField: username,
    passwordField: password,
    onSuccessRedirectTo: "/",
    onFailRedirectTo: "/login"
  }
}
```

Special components: `SignupForm`, `LoginForm`, `LogoutButton`

### Permissions

Control access at the route and query level:

- `IsAuth` - User must be authenticated
- `OwnsRecord` - User must own the record being accessed (checks foreign key relationships)

### Client Dependencies

Include npm packages in your generated client:

```thoth
app MyApp {
  title: "My App",
  clientDep: [
    ("axios", "^1.4.0"),
    ("date-fns", "^2.30.0")
  ]
}
```

## Getting Started

### Prerequisites

Ensure you have the following installed:

- **OCaml** (>= 4.0)
- **Dune** (>= 3.6) - OCaml build system
- **Node.js** (>= 16) - For running generated code
- **Yarn** - Package manager
- **PostgreSQL** (>= 14.7) - Database server

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/abdufelsayed/thoth.git
   cd thoth
   ```

2. **Install OCaml dependencies:**
   ```bash
   opam install . --deps-only
   ```

3. **Build the compiler:**
   ```bash
   dune build
   ```

### Compiling Your First App

1. **Create a `.thoth` file** (or use an example):
   ```bash
   # Use the provided todo app example
   cat examples/todo.thoth
   ```

2. **Compile the application:**
   ```bash
   dune exec -- bin/main.exe examples/todo.thoth my_database
   ```
   
   This generates two directories in `.out/`:
   - `.out/server/` - Node.js + Express backend
   - `.out/client/` - React + Vite frontend

3. **Optional flags:**
   ```bash
   dune exec -- bin/main.exe examples/todo.thoth my_database \
     --output_dir=./output \
     --server_port=3000
   ```
   
   - `--output_dir` (or `-o`) - Output directory (default: `.out`)
   - `--server_port` - Backend port (default: `4000`)

### Running the Generated Application

1. **Start PostgreSQL:**
   ```bash
   # Make sure PostgreSQL is running on your machine
   psql -U postgres -c "SELECT 1"
   ```

2. **Set up the database:**
   ```bash
   cd .out/server
   yarn install
   yarn prisma migrate dev --name init
   ```

3. **Start the backend:**
   ```bash
   yarn dev
   # Server runs on http://localhost:4000
   ```

4. **Start the frontend** (in a new terminal):
   ```bash
   cd .out/client
   yarn install
   yarn dev
   # Client runs on http://localhost:5173
   ```

5. **Open your browser:**
   Navigate to `http://localhost:5173` and start using your app!

## Example Applications

The `examples/` directory contains three complete applications:

### 1. **Todo App** (`examples/todo.thoth`)
A simple task manager with:
- User authentication
- Task creation and deletion
- Real-time task updates
- Personal task lists

### 2. **Chat Room** (`examples/chat.thoth`)
A real-time chat application with:
- User presence indicators (online/offline)
- Message history
- Real-time message delivery
- User-specific message styling

### 3. **Kanban Board** (`examples/kanban.thoth`)
A project management board featuring:
- Drag-and-drop task management
- Multiple columns (Todo, Doing, Done)
- Integration with external libraries (`react-beautiful-dnd`)
- Real-time board updates across clients

Each example demonstrates different aspects of Thoth's capabilities.

## Project Structure

```
thoth/
├── bin/
│   └── main.ml              # Compiler entry point
├── src/
│   ├── analyzer/
│   │   ├── ast/             # Abstract Syntax Tree definitions
│   │   ├── error_handler/   # Error reporting
│   │   ├── parsing/         # Lexer and parser (Menhir)
│   │   └── type_checker/    # Type checking and validation
│   ├── specs/               # Intermediate specifications
│   └── generator/           # Code generation (TypeScript, Prisma)
├── templates/               # Jinja2 templates for code generation
│   ├── client/              # React/TypeScript templates
│   ├── server/              # Express/TypeScript templates
│   └── db/                  # Prisma schema templates
├── examples/                # Example applications
└── language_tools/
    └── thoth-syntax-highlighting/  # VS Code extension
```

### Architecture Overview

- **Lexer** (`src/analyzer/parsing/lexer.mll`) - Tokenizes `.thoth` files
- **Parser** (`src/analyzer/parsing/parser.mly`) - Builds AST using Menhir
- **Type Checker** (`src/analyzer/type_checker/`) - Validates models, queries, components
- **Spec Generator** (`src/specs/`) - Creates intermediate representations
- **Code Generator** (`src/generator/`) - Produces TypeScript and Prisma code
- **Templates** (`templates/`) - Jinja2 templates for clean code generation

## Learn More

### Full Dissertation

For an in-depth understanding of Thoth's design, implementation, and evaluation, read the full dissertation:

📄 [**A DSL for Multitier Web Development**](https://github.com/abdufelsayed/thoth-dissertation/blob/main/output/dissertation.pdf)

The dissertation covers:
- Motivation and related work
- Language design and syntax
- Type system and semantics
- Compiler implementation
- Code generation strategies
- Evaluation and case studies

### VS Code Extension

Syntax highlighting is available for VS Code:

```bash
cd language_tools/thoth-syntax-highlighting
code --install-extension .
```

## Acknowledgments

Thoth was created as a master's dissertation project at the University of Birmingham under the supervision of [Dr. Vincent Rahli](https://www.birmingham.ac.uk/staff/profiles/computer-science/academic-staff/rahli-vincent.aspx).

**Author:** Abdullah Elsayed  
**Repository:** [https://github.com/abdufelsayed/thoth](https://github.com/abdufelsayed/thoth)  
**License:** MIT

---

<p align="center">
  <sub>Made with ❤️ as an academic proof-of-concept for multitier web development</sub>
</p>

