---
audience: [design, engineering, product]
type: specification
tags: [design-system, ui, components, style-guide]
last_updated: 2026-03-30
---

# Design System - Alejandria

> **Para Design Team y Frontend Engineers** - Sistema de diseño completo
> 
> **NOTA**: Este diseño system está planeado para **Fase 2 (Post-MVP)**. El MVP se enfoca en backend + MCP sin frontend web.

---

## 🎨 Principios de Diseño

### 1. Speed First
La búsqueda debe sentirse **instantánea**. Cada interacción debe responder en < 100ms percibidos.

### 2. Minimal Friction
Crear un documento debe tomar **< 3 clicks**. Eliminar pasos innecesarios.

### 3. Context-Aware
Mostrar información relevante sin saturar. El contexto correcto en el momento correcto.

### 4. Keyboard-Friendly
Power users deben poder hacer todo sin mouse. Shortcuts para acciones comunes.

### 5. Accessible by Default
WCAG 2.1 AA compliance. Diseñar para todos desde el inicio.

---

## 🎨 Fundamentos

### Colores

#### Primary Palette
```css
--primary-50:  #EEF2FF;  /* Backgrounds */
--primary-100: #E0E7FF;  /* Hover states */
--primary-200: #C7D2FE;  /* Borders */
--primary-300: #A5B4FC;  /* Disabled */
--primary-400: #818CF8;  /* Secondary actions */
--primary-500: #6366F1;  /* Primary actions */
--primary-600: #4F46E5;  /* Primary hover */
--primary-700: #4338CA;  /* Primary active */
--primary-800: #3730A3;  /* Dark mode primary */
--primary-900: #312E81;  /* Dark mode hover */
```

**Uso:**
- `primary-500`: Botones principales, links, iconos activos
- `primary-600`: Hover de botones principales
- `primary-100`: Backgrounds de elementos seleccionados
- `primary-200`: Borders de inputs focus

#### Neutral Palette
```css
--neutral-50:  #FAFAFA;  /* Page background */
--neutral-100: #F5F5F5;  /* Card background */
--neutral-200: #E5E5E5;  /* Borders */
--neutral-300: #D4D4D4;  /* Disabled text */
--neutral-400: #A3A3A3;  /* Placeholder text */
--neutral-500: #737373;  /* Secondary text */
--neutral-600: #525252;  /* Body text */
--neutral-700: #404040;  /* Headings */
--neutral-800: #262626;  /* Dark mode background */
--neutral-900: #171717;  /* Dark mode cards */
```

#### Semantic Colors
```css
/* Success */
--success-500: #10B981;  /* Green */
--success-600: #059669;  /* Green hover */

/* Warning */
--warning-500: #F59E0B;  /* Amber */
--warning-600: #D97706;  /* Amber hover */

/* Error */
--error-500: #EF4444;    /* Red */
--error-600: #DC2626;    /* Red hover */

/* Info */
--info-500: #3B82F6;     /* Blue */
--info-600: #2563EB;     /* Blue hover */
```

---

### Tipografía

#### Font Families
```css
--font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
--font-mono: 'JetBrains Mono', 'Fira Code', 'Courier New', monospace;
```

#### Type Scale
```css
/* Headings */
--text-4xl: 2.25rem;  /* 36px - Page titles */
--text-3xl: 1.875rem; /* 30px - Section titles */
--text-2xl: 1.5rem;   /* 24px - Card titles */
--text-xl:  1.25rem;  /* 20px - Subsections */
--text-lg:  1.125rem; /* 18px - Large body */

/* Body */
--text-base: 1rem;    /* 16px - Body text */
--text-sm:   0.875rem;/* 14px - Small text */
--text-xs:   0.75rem; /* 12px - Captions */
```

#### Font Weights
```css
--font-normal:   400;
--font-medium:   500;
--font-semibold: 600;
--font-bold:     700;
```

#### Line Heights
```css
--leading-tight:  1.25;  /* Headings */
--leading-normal: 1.5;   /* Body text */
--leading-relaxed: 1.75; /* Long-form content */
```

---

### Spacing

**Sistema de 4px:**
```css
--space-1:  0.25rem;  /* 4px */
--space-2:  0.5rem;   /* 8px */
--space-3:  0.75rem;  /* 12px */
--space-4:  1rem;     /* 16px */
--space-5:  1.25rem;  /* 20px */
--space-6:  1.5rem;   /* 24px */
--space-8:  2rem;     /* 32px */
--space-10: 2.5rem;   /* 40px */
--space-12: 3rem;     /* 48px */
--space-16: 4rem;     /* 64px */
--space-20: 5rem;     /* 80px */
```

**Uso común:**
- `space-2`: Padding interno de botones pequeños
- `space-4`: Padding de cards, gap entre elementos
- `space-6`: Margin entre secciones
- `space-8`: Padding de containers
- `space-12`: Margin entre bloques grandes

---

### Border Radius
```css
--radius-sm:   0.25rem;  /* 4px - Badges */
--radius-md:   0.5rem;   /* 8px - Buttons, inputs */
--radius-lg:   0.75rem;  /* 12px - Cards */
--radius-xl:   1rem;     /* 16px - Modals */
--radius-full: 9999px;   /* Circular */
```

---

### Shadows
```css
--shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
--shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
--shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
--shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1);
```

---

## 🧩 Componentes

### Buttons

#### Primary Button
```jsx
<button className="btn-primary">
  Search Documents
</button>
```

**Styles:**
```css
.btn-primary {
  background: var(--primary-500);
  color: white;
  padding: var(--space-3) var(--space-6);
  border-radius: var(--radius-md);
  font-weight: var(--font-medium);
  transition: all 150ms ease;
}

.btn-primary:hover {
  background: var(--primary-600);
  transform: translateY(-1px);
  box-shadow: var(--shadow-md);
}

.btn-primary:active {
  transform: translateY(0);
}
```

#### Secondary Button
```jsx
<button className="btn-secondary">
  Cancel
</button>
```

**Styles:**
```css
.btn-secondary {
  background: var(--neutral-100);
  color: var(--neutral-700);
  border: 1px solid var(--neutral-200);
  /* ... mismo padding y radius */
}
```

#### Ghost Button
```jsx
<button className="btn-ghost">
  <IconEdit /> Edit
</button>
```

---

### Inputs

#### Text Input
```jsx
<input 
  type="text"
  placeholder="Search documentation..."
  className="input-text"
/>
```

**Styles:**
```css
.input-text {
  width: 100%;
  padding: var(--space-3) var(--space-4);
  border: 1px solid var(--neutral-200);
  border-radius: var(--radius-md);
  font-size: var(--text-base);
  transition: border-color 150ms ease;
}

.input-text:focus {
  outline: none;
  border-color: var(--primary-500);
  box-shadow: 0 0 0 3px var(--primary-100);
}

.input-text::placeholder {
  color: var(--neutral-400);
}
```

#### Search Input
```jsx
<div className="search-input">
  <IconSearch />
  <input type="text" placeholder="Search..." />
  <kbd>/</kbd>
</div>
```

---

### Cards

#### Document Card
```jsx
<div className="card-document">
  <div className="card-header">
    <span className="badge">ADR</span>
    <span className="card-meta">Updated 2 days ago</span>
  </div>
  <h3 className="card-title">ADR-0002: Qdrant for Vector Search</h3>
  <p className="card-snippet">
    We need semantic search with embeddings...
  </p>
  <div className="card-footer">
    <button className="btn-ghost">View</button>
    <span className="card-related">Related: 5</span>
  </div>
</div>
```

**Styles:**
```css
.card-document {
  background: white;
  border: 1px solid var(--neutral-200);
  border-radius: var(--radius-lg);
  padding: var(--space-6);
  transition: all 150ms ease;
}

.card-document:hover {
  border-color: var(--primary-200);
  box-shadow: var(--shadow-md);
  transform: translateY(-2px);
}
```

---

### Badges

```jsx
<span className="badge badge-primary">ADR</span>
<span className="badge badge-success">Active</span>
<span className="badge badge-neutral">Draft</span>
```

**Styles:**
```css
.badge {
  display: inline-flex;
  align-items: center;
  padding: var(--space-1) var(--space-3);
  border-radius: var(--radius-sm);
  font-size: var(--text-xs);
  font-weight: var(--font-medium);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.badge-primary {
  background: var(--primary-100);
  color: var(--primary-700);
}
```

---

### Navigation

#### Sidebar
```jsx
<nav className="sidebar">
  <div className="sidebar-header">
    <Logo />
    <button className="btn-ghost">
      <IconMenu />
    </button>
  </div>
  
  <div className="sidebar-section">
    <h4 className="sidebar-title">Quick Access</h4>
    <a href="/search" className="sidebar-link active">
      <IconSearch /> Search
    </a>
    <a href="/documents" className="sidebar-link">
      <IconDocument /> Documents
    </a>
  </div>
</nav>
```

---

### Modals

```jsx
<div className="modal-overlay">
  <div className="modal">
    <div className="modal-header">
      <h2>Create New Document</h2>
      <button className="btn-ghost">
        <IconClose />
      </button>
    </div>
    
    <div className="modal-body">
      {/* Content */}
    </div>
    
    <div className="modal-footer">
      <button className="btn-secondary">Cancel</button>
      <button className="btn-primary">Create</button>
    </div>
  </div>
</div>
```

---

## ⌨️ Keyboard Shortcuts

### Global
- `/` - Focus search
- `Cmd+K` - Command palette
- `Cmd+N` - New document
- `Esc` - Close modal/sidebar

### Search Results
- `↑↓` - Navigate results
- `Enter` - Open selected
- `Cmd+Enter` - Open in new tab

### Document View
- `E` - Edit document
- `Cmd+S` - Save
- `Cmd+/` - Toggle sidebar

---

## 📱 Responsive Breakpoints

```css
--breakpoint-sm: 640px;   /* Mobile */
--breakpoint-md: 768px;   /* Tablet */
--breakpoint-lg: 1024px;  /* Desktop */
--breakpoint-xl: 1280px;  /* Large desktop */
```

---

## 🎯 Iconografía

**Librería**: Lucide React

**Tamaños:**
```css
--icon-sm: 16px;
--icon-md: 20px;
--icon-lg: 24px;
--icon-xl: 32px;
```

**Iconos comunes:**
- Search: `<IconSearch />`
- Document: `<IconFileText />`
- Edit: `<IconEdit />`
- Delete: `<IconTrash />`
- Settings: `<IconSettings />`
- User: `<IconUser />`

---

## 🌗 Dark Mode

**Variables:**
```css
[data-theme="dark"] {
  --bg-primary: var(--neutral-900);
  --bg-secondary: var(--neutral-800);
  --text-primary: var(--neutral-100);
  --text-secondary: var(--neutral-400);
  --border: var(--neutral-700);
}
```

---

## 📦 Implementación

### TailwindCSS Config
```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: { /* ... */ },
        neutral: { /* ... */ },
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace'],
      },
    },
  },
}
```

### Figma
**Link**: [Figma Design System](#)

---

**Última actualización**: 2026-03-30  
**Mantenido por**: Design Team
