# Database (SQLite)

This workspace includes helper scripts for working with SQLite during development.

Important clarification:
- The Todo FastAPI backend manages its own SQLite database file at:
  simple-todo-application-188632-188641/todo_backend/todo.db
- This app database is auto-initialized on backend startup. You do NOT need to manually initialize a database to run the app.

What’s in this folder:
- init_db.py: Creates a sample SQLite DB (myapp.db) and writes db_connection.txt for local exploration.
- db_shell.py: Interactive SQLite shell helper for the sample DB (myapp.db).
- backup_db.sh / restore_db.sh: Generic backup/restore utilities that operate on the sample DB by default (myapp.db).
- db_visualizer/: Minimal DB viewer utilities (supports multiple engines) for exploration.

Notes on which DB the app actually uses:
- The running Todo app (FastAPI) uses todo_backend/todo.db (inside the backend container/workspace).
- The myapp.db in this folder is for exploration and examples; it is not used by the app unless you explicitly wire it up.

Optional: Inspect the application database
You can inspect the application DB todo_backend/todo.db without modifying it. Choose one of the methods below. Execute one SQL statement at a time.

1) Using sqlite3 (recommended for quick read-only checks):
   - Command format: sqlite3 "<path-to-db>" "SQL_STATEMENT"
   - Examples:
     - Health check (via API):
       curl -s http://localhost:3001/ | jq .
     - Recent todos:
       sqlite3 "simple-todo-application-188632-188641/todo_backend/todo.db" "SELECT id,title,completed,created_at,updated_at FROM todos ORDER BY id DESC LIMIT 10;"
     - List tables:
       sqlite3 "simple-todo-application-188632-188641/todo_backend/todo.db" "SELECT name FROM sqlite_master WHERE type='table' AND name NOT LIKE 'sqlite_%';"
     - Show schema for a table (replace todos with your table name):
       sqlite3 "simple-todo-application-188632-188641/todo_backend/todo.db" ".schema todos"

2) Using the provided db_shell.py (works by default with sample myapp.db, can be pointed to the app DB):
   - Default (sample DB): python database/db_shell.py
   - To point it at the app DB temporarily, run from the database folder:
     - Copy the app DB (recommended to avoid locking): 
       cp ../simple-todo-application-188632-188641/todo_backend/todo.db tmp_app_copy.db
       SQLITE_DB=tmp_app_copy.db python db_shell.py
     - Inside the shell you can run:
       .tables
       .schema
       SELECT ...;
   - Always run one statement at a time. The shell supports:
     .help, .tables, .schema [table], .describe [table], .quit

Verifying data persistence across backend restarts
- Add or modify a todo via the app.
- Stop and restart the backend.
- Re-open todo_backend/todo.db (via sqlite3 or the shell copy approach above) and confirm your changes persist.
  Example check:
  sqlite3 "simple-todo-application-188632-188641/todo_backend/todo.db" "SELECT COUNT(*) FROM todos;"

Back up and restore
- The provided scripts backup_db.sh and restore_db.sh target the sample DB (myapp.db) by default.
- They do NOT operate on the app DB (todo_backend/todo.db) unless you explicitly copy/rename files.
- To back up the app DB manually:
  cp simple-todo-application-188632-188641/todo_backend/todo.db todo_backend_backup_$(date +%Y%m%d%H%M%S).db
- To restore the app DB manually:
  cp todo_backend_backup_TIMESTAMP.db simple-todo-application-188632-188641/todo_backend/todo.db

Cautions
- Avoid writing directly to the live app DB while the backend is running to prevent locks/corruption.
- If you need to explore data interactively, copy the DB and work on the copy.
- Always execute SQL statements one at a time when using sqlite3 or db_shell.py.
