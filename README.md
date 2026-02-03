# beautifyco.de

A modern, Vue 3-powered text formatting and utility toolkit accessible at [beautifyco.de](https://beautifyco.de/).

## 🚀 Overview

This is a client-side web application that provides essential text formatting and encoding/decoding utilities for developers. Built with Vue 3 and Vite, it offers a clean, responsive interface with real-time processing capabilities.

## 🛠️ Features

### Core Formatters
- **JSON Formatter**: Format, validate, and minify JSON with customizable indentation (2-space, 4-space, or minified)
- **Base64 Encoder/Decoder**: Bidirectional Base64 encoding and decoding
- **JWT Decoder**: Decode JWT tokens showing header, payload, and signature sections
- **UUID Generator**: Generate multiple UUID v4 identifiers with copy functionality

### User Experience
- **Quick Switcher**: Ctrl+Space to access any formatter instantly
- **Keyboard Shortcuts**: Ctrl+Enter to process content in most formatters
- **Real-time Processing**: Instant feedback and validation
- **Responsive Design**: Optimized for desktop and mobile devices
- **Modern UI**: Clean, accessible interface with proper error handling

## 🏗️ Technical Architecture

### Technology Stack
- **Frontend Framework**: Vue 3 (Composition API ready)
- **Build Tool**: Vite 6.1.0
- **Development Tools**: Vue DevTools plugin
- **Styling**: Vanilla CSS with modern design tokens
- **Deployment**: Static site generation to `deploy` branch

### Project Structure
```
src/
├── App.vue                 # Main application with search interface
├── main.js                # Application entry point
└── formatters/            # Formatter components
    ├── JsonFormatter.vue   # JSON processing with advanced features
    ├── Base64Encoder.vue   # Base64 encoding functionality
    ├── Base64Decoder.vue   # Base64 decoding functionality
    ├── JwtDecoder.vue      # JWT token parsing and display
    └── UuidGenerator.vue   # UUID generation utilities
```

### Component Architecture
Each formatter follows a consistent pattern:
- **Side-by-side Layout**: Input and output panels with responsive grid
- **Keyboard Navigation**: Consistent shortcuts across components
- **Error Handling**: Graceful error states with user-friendly messages
- **Accessibility**: Proper ARIA labels and keyboard support

### Key Implementation Details

#### JSON Formatter (`JsonFormatter.vue`)
- **Advanced Rendering**: Custom HTML renderer for syntax highlighting
- **Indentation Control**: Segmented buttons (2sp/4sp/Min) replacing dropdowns
- **Collapsible Structure**: Interactive arrows for nested JSON objects/arrays
- **Line Numbers**: CSS counter-based numbering system
- **Performance**: Optimized for large JSON documents

#### Search Interface (`App.vue`)
- **Fuzzy Search**: Search by name, ID, or numeric index
- **Dynamic Filtering**: Real-time results as you type
- **Component Registry**: Centralized formatter management
- **State Management**: Persistent selection across sessions

### Modern Features
- **ES6 Modules**: Modern JavaScript with proper imports
- **CSS Custom Properties**: Design token system for consistent theming
- **Grid Layout**: Responsive layouts using CSS Grid
- **Progressive Enhancement**: Works without JavaScript for basic functionality

## 🎨 Design System

### Color Palette
- **Primary**: Modern blue (#3b82f6) with hover states (#4338ca)
- **Neutral Grays**: Tailwind-inspired scale from #f8fafc to #1e293b
- **Success States**: Green indicators for copy operations
- **Error States**: Red messaging with accessible contrast ratios

### Typography
- **Interface**: System fonts (-apple-system, BlinkMacSystemFont, Segoe UI)
- **Code**: Monospace stack (Monaco, Menlo, Ubuntu Mono)
- **Responsive Sizing**: Scales appropriately across device sizes

## 🚀 Development

### Prerequisites
- Node.js (Latest LTS recommended)
- npm or yarn package manager

### Getting Started
```bash
# Clone the repository
git clone https://github.com/vishnugt/beautifyco.de.git
cd beautifyco.de

# Install dependencies
npm install

# Start development server
npm run dev
# Server runs on http://localhost:5173

# Build for production
npm run build

# Preview production build
npm run preview
```

### Development Scripts
- `npm run dev` - Start Vite dev server with hot reload
- `npm run build` - Build production bundle
- `npm run preview` - Preview production build locally

### Project Configuration
- **Vite Config**: Modern build tooling with Vue 3 support
- **JSConfig**: JavaScript project configuration for better IDE support
- **Vue DevTools**: Integrated development tools for debugging

## 🔧 Adding New Formatters

To add a new formatter, follow this pattern:

1. **Create Component**: Add new `.vue` file in `src/formatters/`
2. **Register Component**: Import and register in `App.vue`
3. **Add to Registry**: Update the `scripts` array with formatter metadata
4. **Follow Conventions**: Use consistent styling and keyboard shortcuts

### Example Formatter Template
```vue
<template>
  <div class="formatter-container">
    <div class="side-by-side-layout">
      <div class="input-side">
        <textarea v-model="input" @keydown="handleInputKeydown" />
      </div>
      <div class="output-side" v-if="output">
        <textarea v-model="output" readonly />
      </div>
      <div class="empty-state" v-else>
        <p>Processed output will appear here</p>
      </div>
    </div>
    <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
  </div>
</template>
```

Sample implementation reference: [Add Formatter PR](https://github.com/vishnugt/beautifyco.de/commit/434ed7e2bfb7f5a3c851ebeb47eb9a7154fe8e6b)

## 📱 Responsive Design

The application is fully responsive with breakpoints at:
- **Desktop**: 1024px+ (side-by-side layout)
- **Tablet**: 768px-1024px (adapted spacing)
- **Mobile**: <768px (stacked layout)

### Mobile Optimizations
- Stacked input/output layout
- Touch-friendly button sizing
- Optimized keyboard handling
- Reduced content density

## ♿ Accessibility

- **Keyboard Navigation**: Full keyboard support for all interactions
- **Screen Readers**: Proper ARIA labels and semantic HTML
- **Color Contrast**: WCAG AA compliant contrast ratios
- **Focus Management**: Visible focus indicators throughout
- **Error Messaging**: Clear, actionable error descriptions

## 🔐 Security

- **Client-Side Only**: No data transmission to external servers
- **Input Validation**: Proper sanitization and validation
- **XSS Prevention**: Safe HTML rendering practices
- **Content Security**: No eval() or unsafe dynamic code execution

## 📊 Performance

- **Bundle Size**: Optimized Vite build with tree shaking
- **Lazy Loading**: Components loaded on-demand
- **Memory Management**: Proper cleanup in Vue lifecycle
- **Rendering**: Efficient virtual DOM updates

## 🚀 Deployment

The application deploys to a static `deploy` branch for GitHub Pages:

```bash
npm run build
# Built files are committed to deploy branch
# Auto-deployment via GitHub Actions
```

## 🤝 Contributing

1. Fork the repository
2. Create feature branch: `git checkout -b feature-name`
3. Follow existing code patterns and conventions
4. Ensure responsive design across all screen sizes
5. Test keyboard navigation and accessibility
6. Submit PR with clear description

## 📄 License

This project is open source and available under the MIT License.

---

**Built with ❤️ using Vue 3 and modern web technologies**