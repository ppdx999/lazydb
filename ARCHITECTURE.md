# LazyDB Architecture Documentation

## Project Overview

LazyDB is a cross-platform Terminal User Interface (TUI) database management tool written in Rust. It provides an intuitive keyboard-driven interface for managing MySQL, PostgreSQL, and SQLite databases. The project is a fork of the gobang project, redesigned for ease of use and efficient database operations.

### Key Features
- Cross-platform support (macOS, Windows, Linux)
- Multiple database support (MySQL, PostgreSQL, SQLite)
- Intuitive keyboard-only control
- Async database operations
- Configurable key bindings
- Tab-based interface for different views

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     LazyDB Application                          │
├─────────────────────────────────────────────────────────────────┤
│                    Presentation Layer                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐│
│  │ Connections │ │ Databases   │ │ Table View  │ │ SQL Editor  ││
│  │ Component   │ │ Component   │ │ Component   │ │ Component   ││
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘│
├─────────────────────────────────────────────────────────────────┤
│                    Application Layer                            │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              App (Main Controller)                          ││
│  │  - Focus Management    - Event Routing                     ││
│  │  - State Coordination  - UI Layout                         ││
│  └─────────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────┤
│                   Business Logic Layer                          │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐│
│  │ Config      │ │ Event       │ │ Database    │ │ Components  ││
│  │ Management  │ │ Handling    │ │ Abstraction │ │ Logic       ││
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘│
├─────────────────────────────────────────────────────────────────┤
│                     Data Layer                                  │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                │
│  │   MySQL     │ │ PostgreSQL  │ │   SQLite    │                │
│  │    Pool     │ │    Pool     │ │    Pool     │                │
│  └─────────────┘ └─────────────┘ └─────────────┘                │
├─────────────────────────────────────────────────────────────────┤
│                 Infrastructure Layer                            │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐│
│  │   SQLx      │ │ Crossterm   │ │    TUI      │ │   Tokio     ││
│  │ (Database)  │ │ (Terminal)  │ │ (Rendering) │ │  (Async)    ││
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

## Module Structure

```
src/
├── main.rs                 # Application entry point
├── app.rs                  # Main application controller
├── config.rs               # Configuration management
├── clipboard.rs            # System clipboard operations
├── log.rs                  # Logging infrastructure
└── event/
    ├── mod.rs              # Event system module
    ├── events.rs           # Event handling and processing
    └── key.rs              # Keyboard input definitions
└── database/
    ├── mod.rs              # Database module exports
    ├── mysql.rs            # MySQL implementation
    ├── postgres.rs         # PostgreSQL implementation
    └── sqlite.rs           # SQLite implementation
└── components/
    ├── mod.rs              # Component module exports
    ├── command.rs          # Command definitions for help
    ├── completion.rs       # Auto-completion component
    ├── connections.rs      # Database connections panel
    ├── databases.rs        # Database/table tree view
    ├── error.rs            # Error display component
    ├── help.rs             # Help panel component
    ├── properties.rs       # Object properties panel
    ├── record_table.rs     # Table data display
    ├── sql_editor.rs       # SQL query editor
    ├── tab.rs              # Tab management
    ├── table.rs            # Generic table component
    ├── table_filter.rs     # Table filtering
    ├── table_status.rs     # Table status information
    ├── table_value.rs      # Individual cell values
    ├── database_filter.rs  # Database filtering
    └── utils/
        └── scroll_vertical.rs  # Vertical scrolling utility
└── ui/
    └── reflow.rs           # Text reflow and wrapping
```

## Core Components

### 1. Application Controller (`app.rs`)

The `App` struct serves as the main application controller, coordinating all UI components and managing application state.

```rust
pub struct App {
    // UI Components
    record_table: RecordTableComponent,
    properties: PropertiesComponent,
    sql_editor: SqlEditorComponent,
    tab: TabComponent,
    help: HelpComponent,
    databases: DatabasesComponent,
    connections: ConnectionsComponent,
    error: ErrorComponent,
    
    // Application State
    focus: Focus,
    pool: Option<Box<dyn Pool>>,
    left_main_chunk_percentage: u16,
    config: Config,
}
```

**Responsibilities:**
- Focus management between panels
- Event routing to appropriate components
- Database connection lifecycle management
- UI layout coordination
- State synchronization

### 2. Focus Management

```rust
pub enum Focus {
    DabataseList,    // Database/table tree view
    Table,           // Table data view
    ConnectionList,  // Connection management panel
}
```

Focus determines which component receives keyboard input and is highlighted in the UI.

### 3. Database Abstraction Layer

The database layer uses a trait-based approach for supporting multiple database types:

```rust
#[async_trait]
pub trait Pool: Send + Sync {
    async fn execute(&self, query: &str) -> anyhow::Result<ExecuteResult>;
    async fn get_databases(&self) -> anyhow::Result<Vec<Database>>;
    async fn get_tables(&self, database: &str) -> anyhow::Result<Vec<DatabseTreeItem>>;
    async fn get_records(&self, query: &RecordsQuery) -> anyhow::Result<(Vec<Box<dyn TableRow>>, Vec<String>)>;
    async fn get_columns(&self, table: &Table) -> anyhow::Result<(Vec<Box<dyn TableRow>>, Vec<String>)>;
    async fn close(&self);
}
```

**Implementations:**
- `MySqlPool` - MySQL/MariaDB support via SQLx
- `PostgresPool` - PostgreSQL support via SQLx  
- `SqlitePool` - SQLite support via SQLx

## Data Flow and Interaction Patterns

### Event Flow
```
User Input (Keyboard) 
    ↓
Crossterm Event Capture
    ↓
Event Processing (events.rs)
    ↓
App Event Handler (app.rs)
    ↓
Component Event Routing
    ↓
Component State Updates
    ↓
UI Re-rendering (TUI)
```

### Database Query Flow
```
User Action (e.g., select table)
    ↓
Component Event Handler
    ↓
App Controller
    ↓
Database Pool (async)
    ↓
SQLx Query Execution
    ↓
Result Processing
    ↓
Component State Update
    ↓
UI Refresh
```

## UI Component Hierarchy

### Layout Structure
```
┌─────────────────────────────────────────────────────────────────┐
│                        Help Bar                                 │
├─────────────────┬───────────────────────────────────────────────┤
│                 │                                               │
│   Connections   │              Main Content Area                │
│     Panel       │  ┌─────────────┬─────────────────────────────┐│
│                 │  │ Database/   │                             ││
│  ┌─────────────┐│  │ Table Tree  │      Table Data View        ││
│  │ Connection  ││  │             │                             ││
│  │ List        ││  └─────────────┼─────────────────────────────┤│
│  └─────────────┘│  │ Properties  │                             ││
│                 │  │ Panel       │                             ││
│                 │  └─────────────┴─────────────────────────────┘│
├─────────────────┼───────────────────────────────────────────────┤
│     Status      │                Tab Bar                        │
└─────────────────┴───────────────────────────────────────────────┘
```

### Component Relationships
- **ConnectionsComponent**: Manages database connections
- **DatabasesComponent**: Displays database/table hierarchy
- **RecordTableComponent**: Shows table data with pagination
- **PropertiesComponent**: Displays selected object properties
- **SqlEditorComponent**: Provides SQL query interface
- **TabComponent**: Manages different views (Records, Columns, etc.)
- **HelpComponent**: Context-sensitive help overlay

## Configuration System

### Configuration Structure
```rust
pub struct Config {
    pub conn: Vec<Connection>,        // Database connections
    pub key_config: KeyConfig,        // Key bindings
    pub log_level: LogLevel,          // Logging configuration
}

pub struct KeyConfig {
    // Navigation keys
    pub scroll_up: Key,
    pub scroll_down: Key,
    pub focus_left: Key,
    pub focus_right: Key,
    pub toggle_main_panels: Key,      // Tab key functionality
    
    // Action keys
    pub enter: Key,
    pub copy: Key,
    pub filter: Key,
    pub open_help: Key,
    
    // Tab keys for different views
    pub tab_records: Key,
    pub tab_columns: Key,
    pub tab_sql_editor: Key,
    // ... more tab bindings
}
```

### Configuration Files
- **Location**: `$HOME/.config/lazydb/config.toml`
- **Format**: TOML with connection definitions and key bindings
- **Default fallback**: Sensible defaults for all configurations

## Event Handling System

### Event Processing Pipeline

1. **Input Capture** (`events.rs`)
   ```rust
   pub struct Events {
       rx: mpsc::Receiver<Event<Key>>,
       input_handle: thread::JoinHandle<()>,
   }
   ```

2. **Event Types** (`key.rs`)
   ```rust
   pub enum Key {
       Enter, Tab, Backspace, Esc,
       Left, Right, Up, Down,
       Char(char), Ctrl(char), Alt(char),
       F(u8), // Function keys
       // ... more key types
   }
   ```

3. **Event Routing** (`app.rs`)
   - Error component gets first priority
   - Help component (when visible)
   - Focus-based component routing
   - App-level focus management

### Async Operations
- Database queries run asynchronously using Tokio
- UI remains responsive during long-running queries
- Background tasks for connection management

## Key Data Structures

### Database Entities
```rust
pub struct Database {
    pub name: String,
    pub tables: Vec<Table>,
}

pub struct Table {
    pub name: String,
    pub database: String,
    pub table_type: String,
}

pub trait TableRow: Debug + Send + Sync {
    fn fields(&self) -> Vec<String>;
    fn columns(&self) -> Vec<String>;
}
```

### UI State Management
```rust
pub struct TableComponent {
    pub headers: Vec<String>,
    pub rows: Vec<Vec<String>>,
    pub selected_row: usize,
    pub selected_column: usize,
    // ... pagination and selection state
}
```

## External Dependencies

### Core Dependencies
- **sqlx** (0.8.6): Async SQL database driver for MySQL, PostgreSQL, SQLite
- **tokio** (1.47.1): Async runtime for Rust
- **tui** (0.15.0): Terminal UI framework for rendering
- **crossterm** (0.20.0): Cross-platform terminal manipulation
- **anyhow** (1.0.99): Error handling with context
- **serde** (1.0.219): Serialization framework for configuration

### Utility Dependencies
- **chrono** (0.4.41): Date and time handling
- **unicode-width** (0.1.14): Unicode text width calculation
- **itertools** (0.10.5): Iterator utilities
- **rust_decimal** (1.37.2): Decimal number handling for database values

### Development Dependencies
- **pretty_assertions** (1.4.1): Better assertion output in tests
- **structopt** (0.3.26): Command-line argument parsing

## Entry Points and Execution Flow

### Main Entry Point (`main.rs`)
```rust
#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // 1. Parse CLI arguments
    let cli_config = CliConfig::from_args();
    
    // 2. Load configuration
    let config = Config::new(&cli_config)?;
    
    // 3. Initialize logging
    Logger::init(&config.log_level)?;
    
    // 4. Create and run application
    let mut app = App::new(config);
    run_app(&mut app).await
}
```

### Application Lifecycle
1. **Initialization**
   - Parse CLI arguments and configuration
   - Set up logging and error handling
   - Initialize UI components and state

2. **Main Loop** (`run_app` function)
   - Create terminal interface
   - Set up event handling
   - Enter main event loop:
     ```rust
     loop {
         app.draw(&mut terminal)?;
         if let Event::Input(key) = events.next()? {
             if app.event(key).await?.should_quit() {
                 break;
             }
         }
     }
     ```

3. **Shutdown**
   - Close database connections
   - Restore terminal state
   - Clean up resources

### Component Lifecycle
Each UI component follows a consistent pattern:
- **Creation**: Initialize with configuration
- **Event Handling**: Process keyboard input
- **State Updates**: Modify internal state
- **Rendering**: Draw to terminal using TUI framework
- **Cleanup**: Release resources when destroyed

## Architecture Strengths

1. **Modular Design**: Clear separation of concerns between UI, business logic, and data layers
2. **Async Support**: Non-blocking database operations maintain UI responsiveness
3. **Extensible**: Easy to add new database types through the Pool trait
4. **Configurable**: Comprehensive key binding and connection configuration
5. **Cross-platform**: Works consistently across different operating systems

## Areas for Improvement

1. **Error Handling**: Replace unwrap() calls with proper error handling
2. **Testing**: Add comprehensive test coverage, especially for database operations
3. **Documentation**: Add inline documentation and usage examples
4. **Performance**: Optimize string handling and memory allocations
5. **Dependency Updates**: Update TUI framework to latest version (ratatui)

This architecture provides a solid foundation for a terminal-based database management tool, with clear separation of concerns and good extensibility for future enhancements.