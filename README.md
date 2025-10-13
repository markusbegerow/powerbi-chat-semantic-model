# Power BI Chat - Semantic Model Assistant

An AI-powered Power BI External Tool that provides an intelligent chat interface to analyze and interact with your Power BI semantic models in real-time.

<img width="1186" height="793" alt="image" src="https://github.com/user-attachments/assets/61351f14-5df8-4ab9-aa6a-6471112f825e" />

## Overview

Power BI Chat is an Electron-based desktop application that integrates directly with Power BI Desktop as an External Tool. It connects to your semantic model via XMLA endpoints, retrieves metadata and sample data, and uses AI to help you understand and query your data model.

## Features

- **Real-time Semantic Model Access** - Automatically connects to your open Power BI Desktop reports
- **AI-Powered Assistant** - Uses OpenAI or compatible APIs to provide intelligent insights about your data model
- **Complete Model Visibility** - View tables, columns, measures, relationships, and sample data
- **DAX Query Execution** - Execute DAX queries directly from the chat interface
- **Sample Data Preview** - See actual sample data from your tables to understand your data better
- **Connection Monitoring** - Real-time connection status with automatic health checks
- **User-Friendly Interface** - Clean, modern UI with collapsible sections and intuitive navigation

## Prerequisites

Before installing Power BI Chat, ensure you have:

- **Windows OS** (Windows 10 or later)
- **Power BI Desktop** (latest version)
- **.NET Framework 4.8** (for ADOMD.NET bridge)ok this
- **Node.js** (v16 or later) and npm
- **OpenAI API Key** or compatible AI service (e.g., Azure OpenAI, local LLM endpoint)

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/powerbi-chat-semantic-model.git
cd powerbi-chat-semantic-model
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Build the .NET Bridge

The application uses a .NET bridge to connect to Power BI's XMLA endpoint via ADOMD.NET:

```bash
cd XmlaBridge
dotnet build -c Release
cd ..
```

Verify the DLL was created at:
```
XmlaBridge\bin\Release\net48\XmlaBridge.dll
```

### 4. Install as Power BI External Tool

Run the installation script:

```bash
install-tool.bat
```

This copies the `.pbitool.json` file to Power BI's External Tools directory, making it available in Power BI Desktop.

## Configuration

### API Settings

On first launch, configure your AI service:

1. **API URL**: Default is `https://api.openai.com/v1/chat/completions`
   - For Azure OpenAI: `https://your-resource.openai.azure.com/openai/deployments/your-deployment/chat/completions?api-version=2024-02-15-preview`
   - For local LLMs: `http://localhost:11434/v1/chat/completions` (Ollama example)

2. **API Token**: Your API key
   - OpenAI: Get from [platform.openai.com](https://platform.openai.com/api-keys)
   - Azure OpenAI: Get from Azure Portal

3. **Model Name**: Default is `gpt-4`
   - Options: `gpt-4`, `gpt-3.5-turbo`, `gpt-4-turbo`, etc.

Settings are saved in localStorage and persist between sessions.

### Advanced Configuration

Edit `config.js` to customize:

- Window dimensions
- Query timeouts
- Sample data row limits
- .NET bridge path
- XMLA endpoint settings

## Usage

### Starting the Application

#### From Power BI Desktop (Recommended)

1. Open a Power BI Desktop report
2. Navigate to **External Tools** ribbon
3. Click **Power BI Chat**
4. The application will automatically connect to your semantic model

#### Standalone Mode

```bash
npm start
```

Or for development with DevTools:

```bash
npm run dev
```

### Chat Interface

Once connected, you can:

**Ask Questions About Your Model:**
```
What tables are in this model?
Show me the measures related to sales
How is the Customer table related to Sales?
```

**Request DAX Queries:**
```
Write a DAX query to show top 10 customers by revenue
Create a measure for year-over-year growth
```

**Execute DAX Queries:**
```
DAX: EVALUATE TOPN(10, Customer, [Total Revenue])
```

or

```
EXECUTE: SUMMARIZE(Sales, Sales[Category], "Total", SUM(Sales[Amount]))
```

### Understanding the Interface

- **Left Sidebar**: Shows semantic model metadata
  - Tables (with column counts and sample data)
  - Measures (with expressions)
  - Relationships (with cardinality)

- **Main Chat Area**: Interact with the AI assistant

- **Settings Panel**: Configure API credentials and model preferences

- **Connection Status**: Shows real-time connection health
  - Green: Connected
  - Yellow: Connecting
  - Red: Disconnected

## Architecture

```
┌─────────────────────┐
│   Electron UI       │
│   (HTML/CSS/JS)     │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   Main Process      │
│   (Node.js)         │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   electron-edge-js  │
│   (JS ↔ .NET Bridge)│
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   XmlaBridge.dll    │
│   (C# / .NET 4.8)   │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   ADOMD.NET Client  │
│   (Microsoft)       │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   Power BI Desktop  │
│   (XMLA Endpoint)   │
└─────────────────────┘
```

### Key Components

- **main.js**: Electron main process, handles window creation and IPC
- **renderer.js**: UI logic, chat handling, metadata display
- **xmla-connection.js**: XMLA client for querying semantic models
- **dax-validator.js**: Validates and sanitizes DAX queries
- **XmlaBridge/**: C# project for ADOMD.NET integration
- **config.js**: Centralized configuration

## Troubleshooting

### Connection Issues

**Error: "Failed to connect to semantic model"**
- Ensure Power BI Desktop is running
- Verify the report is open (not just Power BI Desktop)
- Check that External Tools are enabled in Power BI settings
- Restart Power BI Desktop

**Error: ".NET Bridge DLL Not Found"**
- Build the XmlaBridge project: `cd XmlaBridge && dotnet build -c Release`
- Verify .NET Framework 4.8 is installed
- Check `config.js` BRIDGE.PATH matches your build output

### API Issues

**Error: "HTTP 401: Unauthorized"**
- Verify your API token is correct
- Check that your API key has proper permissions
- For Azure OpenAI, ensure the deployment name is correct

**Error: "Please configure API URL and Token"**
- Open the settings panel (expand the sidebar)
- Enter your API credentials
- Click save or change focus to persist settings

### Query Issues

**Error: "Query timeout"**
- Increase timeout in `config.js`: `QUERY.COMMAND_TIMEOUT`
- Simplify your DAX query
- Check Power BI Desktop performance

**Error: "Query error: Invalid DAX syntax"**
- Verify DAX syntax
- Check table and column names match exactly
- Use proper DAX functions (EVALUATE, etc.)

## Development

### Running in Development Mode

```bash
npm run dev
```

This opens DevTools automatically for debugging.

### File Structure

```
powerbi-chat-semantic-model/
├── main.js                 # Electron main process
├── renderer.js             # UI logic
├── preload.js              # Electron preload script
├── index.html              # Main UI
├── styles.css              # Styling
├── config.js               # Configuration
├── xmla-connection.js      # XMLA client
├── dax-validator.js        # DAX validation
├── utils.js                # Utility functions
├── package.json            # npm dependencies
├── PowerBIChat.pbitool.json # External tool definition
├── install-tool.bat        # Installation script
├── XmlaBridge/             # .NET bridge project
│   ├── XmlaBridge.csproj
│   ├── Startup.cs
│   └── bin/Release/net48/
└── README.md
```

### Building for Production

To create a distributable package:

```bash
# Install electron-builder
npm install --save-dev electron-builder

# Build
npm run build
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

### Guidelines

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the ISC License - see the LICENSE file for details.

## Acknowledgments

- Built with [Electron](https://www.electronjs.org/)
- Uses [electron-edge-js](https://github.com/agracio/electron-edge-js) for .NET interop
- Powered by [Microsoft ADOMD.NET](https://docs.microsoft.com/en-us/analysis-services/adomd/developing-with-adomd-net)
- AI integration via [OpenAI API](https://platform.openai.com/)

## Support

If you find this tool helpful, consider supporting the development:

- [Donate via PayPal](https://paypal.me/MarkusBegerow?country.x=DE&locale.x=de_DE)
- [Star this repository](https://github.com/markusbegerow/powerbi-chat-semantic-model)

## Author

Power BI Chat Team

## Links

- [GitHub Repository](https://github.com/markusbegerow/powerbi-chat-semantic-model)
- [Issues & Bug Reports](https://github.com/markusbegerow/powerbi-chat-semantic-model/issues)
- [Power BI External Tools Documentation](https://docs.microsoft.com/en-us/power-bi/transform-model/desktop-external-tools)

## 📬 Contact

- 🧑‍💻 [Markus Begerow](https://linkedin.com/in/markusbegerow)
- 💾 [GitHub](https://github.com/markusbegerow)
- ✉️ [Twitter](https://x.com/markusbegerow)

---

Made with ❤️ for the Power BI community
