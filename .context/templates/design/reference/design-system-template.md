# Design System Template

## Design System - [Project Name]

**Version**: [X.X]  
**Last Updated**: [YYYY-MM-DD]  
**Design Lead**: [Name]  
**Engineering Lead**: [Name]  

## 1. Overview

### Purpose
[Descripción del propósito del design system y qué problemas resuelve]

### Scope
- **Components**: [Tipos de componentes incluidos]
- **Patterns**: [Patrones de diseño cubiertos]
- **Guidelines**: [Guías incluidas]
- **Tools**: [Herramientas y frameworks]

### Principles
- **Principle 1**: [Descripción y ejemplos]
- **Principle 2**: [Descripción y ejemplos]
- **Principle 3**: [Descripción y ejemplos]

## 2. Visual Identity

### Brand Colors
```css
/* Primary Colors */
--primary-50: #[hex];
--primary-100: #[hex];
--primary-500: #[hex]; /* Main brand color */
--primary-900: #[hex];

/* Secondary Colors */
--secondary-50: #[hex];
--secondary-500: #[hex];

/* Neutral Colors */
--gray-50: #[hex];
--gray-100: #[hex];
--gray-500: #[hex];
--gray-900: #[hex];

/* Semantic Colors */
--success: #[hex];
--warning: #[hex];
--error: #[hex];
--info: #[hex];
```

### Typography
```css
/* Font Families */
--font-sans: [Font Name], sans-serif;
--font-mono: [Font Name], monospace;

/* Font Sizes */
--text-xs: 0.75rem;    /* 12px */
--text-sm: 0.875rem;   /* 14px */
--text-base: 1rem;     /* 16px */
--text-lg: 1.125rem;   /* 18px */
--text-xl: 1.25rem;    /* 20px */
--text-2xl: 1.5rem;    /* 24px */
--text-3xl: 1.875rem;  /* 30px */

/* Font Weights */
--font-light: 300;
--font-normal: 400;
--font-medium: 500;
--font-semibold: 600;
--font-bold: 700;
```

### Spacing
```css
--space-1: 0.25rem;   /* 4px */
--space-2: 0.5rem;    /* 8px */
--space-3: 0.75rem;   /* 12px */
--space-4: 1rem;      /* 16px */
--space-6: 1.5rem;    /* 24px */
--space-8: 2rem;      /* 32px */
--space-12: 3rem;     /* 48px */
--space-16: 4rem;     /* 64px */
```

### Shadows
```css
--shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
--shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
--shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
--shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
```

## 3. Components

### Buttons

#### Primary Button
```jsx
<Button variant="primary" size="md">
  Button Text
</Button>
```

**States:**
- **Default**: [Screenshot/description]
- **Hover**: [Screenshot/description]
- **Active**: [Screenshot/description]
- **Disabled**: [Screenshot/description]
- **Loading**: [Screenshot/description]

**Sizes:**
- **Small**: 32px height
- **Medium**: 40px height  
- **Large**: 48px height

#### Secondary Button
```jsx
<Button variant="secondary" size="md">
  Button Text
</Button>
```

#### Ghost Button
```jsx
<Button variant="ghost" size="md">
  Button Text
</Button>
```

### Form Elements

#### Input Field
```jsx
<Input placeholder="Enter text" />
```

**States:**
- **Default**: [Description]
- **Focus**: [Description]
- **Error**: [Description]
- **Disabled**: [Description]

#### Select
```jsx
<Select>
  <Select.Option value="option1">Option 1</Select.Option>
  <Select.Option value="option2">Option 2</Select.Option>
</Select>
```

#### Checkbox
```jsx
<Checkbox label="Accept terms" />
```

#### Radio Button
```jsx
<RadioGroup>
  <Radio value="option1">Option 1</Radio>
  <Radio value="option2">Option 2</Radio>
</RadioGroup>
```

### Navigation

#### Header Navigation
```jsx
<Header>
  <Logo />
  <Nav>
    <NavItem href="/docs">Documentation</NavItem>
    <NavItem href="/api">API</NavItem>
  </Nav>
  <UserMenu />
</Header>
```

#### Sidebar Navigation
```jsx
<Sidebar>
  <SidebarItem href="/dashboard" icon={DashboardIcon}>
    Dashboard
  </SidebarItem>
  <SidebarItem href="/settings" icon={SettingsIcon}>
    Settings
  </SidebarItem>
</Sidebar>
```

### Data Display

#### Table
```jsx
<Table>
  <TableHeader>
    <TableRow>
      <TableHead>Name</TableHead>
      <TableHead>Status</TableHead>
      <TableHead>Actions</TableHead>
    </TableRow>
  </TableHeader>
  <TableBody>
    {/* Table rows */}
  </TableBody>
</Table>
```

#### Card
```jsx
<Card>
  <CardHeader>
    <CardTitle>Card Title</CardTitle>
  </CardHeader>
  <CardContent>
    Card content goes here
  </CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>
```

#### Badge
```jsx
<Badge variant="success">Active</Badge>
<Badge variant="warning">Pending</Badge>
<Badge variant="error">Error</Badge>
```

### Feedback

#### Toast/Notification
```jsx
<Toast variant="success">
  Operation completed successfully
</Toast>
```

#### Modal
```jsx
<Modal isOpen={isOpen} onClose={onClose}>
  <ModalHeader>
    <ModalTitle>Confirm Action</ModalTitle>
  </ModalHeader>
  <ModalBody>
    Are you sure you want to proceed?
  </ModalBody>
  <ModalFooter>
    <Button variant="ghost" onClick={onClose}>
      Cancel
    </Button>
    <Button variant="primary" onClick={onConfirm}>
      Confirm
    </Button>
  </ModalFooter>
</Modal>
```

## 4. Layout Patterns

### Grid System
```css
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
}

.grid {
  display: grid;
  gap: 1rem;
}

.grid-cols-1 { grid-template-columns: repeat(1, minmax(0, 1fr)); }
.grid-cols-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
.grid-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
.grid-cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
```

### Page Layouts

#### Dashboard Layout
```
┌─────────────────────────────────────┐
│                Header                │
├─────────┬───────────────────────────┤
│         │                           │
│ Sidebar │        Main Content       │
│         │                           │
│         │                           │
└─────────┴───────────────────────────┘
```

#### Document Layout
```
┌─────────────────────────────────────┐
│                Header                │
├─────────────────────────────────────┤
│            Breadcrumb               │
├─────────────────────────────────────┤
│                                     │
│            Page Content             │
│                                     │
├─────────────────────────────────────┤
│              Footer                 │
└─────────────────────────────────────┘
```

### Responsive Breakpoints
```css
/* Mobile First Approach */
@media (min-width: 640px) { /* sm */ }
@media (min-width: 768px) { /* md */ }
@media (min-width: 1024px) { /* lg */ }
@media (min-width: 1280px) { /* xl */ }
@media (min-width: 1536px) { /* 2xl */ }
```

## 5. Interaction Patterns

### Loading States
- **Skeleton Loading**: Para contenido asíncrono
- **Spinner Loading**: Para acciones rápidas
- **Progress Bar**: Para procesos largos

### Empty States
- **No Data**: Cuando no hay contenido
- **Error State**: Cuando algo falla
- **First Use**: Para usuarios nuevos

### Hover & Focus States
- **Hover**: Indicar elementos interactivos
- **Focus**: Accesibilidad keyboard navigation
- **Active**: Feedback de click/touch

## 6. Accessibility Guidelines

### WCAG 2.1 AA Compliance
- **Color Contrast**: Mínimo 4.5:1 para texto normal
- **Focus Indicators**: Visibles y claros
- **Keyboard Navigation**: Todo accesible por teclado
- **Screen Readers**: ARIA labels apropiadas

### Typography Accessibility
- **Font Size**: Mínimo 16px para texto principal
- **Line Height**: 1.5 para mejor legibilidad
- **Text Spacing**: Adeguado para dislexia

### Color Accessibility
- **Don't rely on color alone**: Usar iconos + texto
- **High Contrast Mode**: Soporte para Windows High Contrast
- **Color Blindness**: Testear con simuladores

## 7. Iconography

### Icon Library
- **Library**: [Font Awesome, Heroicons, custom]
- **Size**: 16px, 20px, 24px, 32px
- **Style**: Consistente (outline, filled, etc.)

### Icon Usage Guidelines
- **Meaning**: Iconos con significado claro
- **Labels**: Siempre con texto o aria-label
- **Consistency**: Mismo icon para misma acción

## 8. Animation & Motion

### Principles
- **Purposeful**: Motion con propósito
- **Subtle**: No distractivo
- **Consistent**: Tiempos consistentes

### Timing
```css
/* Duration */
--duration-fast: 150ms;
--duration-normal: 300ms;
--duration-slow: 500ms;

/* Easing */
--ease-in: cubic-bezier(0.4, 0, 1, 1);
--ease-out: cubic-bezier(0, 0, 0.2, 1);
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
```

### Animations
- **Transitions**: Para cambios de estado
- **Entrances**: Para elementos que aparecen
- **Micro-interactions**: Feedback sutil

## 9. Content Guidelines

### Voice & Tone
- **Voice**: [Descripción del voice de la marca]
- **Tone**: [Cómo adaptar el tone]
- **Guidelines**: [Reglas de escritura]

### Text Patterns
- **Headings**: H1-H6 hierarchy
- **Body Text**: Line length 50-75 characters
- **Links**: Descriptive text, no "click here"

### Error Messages
- **Clear**: Explican qué pasó
- **Helpful**: Sugieren qué hacer
- **Polite**: No culpan al usuario

## 10. Implementation Guidelines

### Technology Stack
- **CSS Framework**: [Tailwind, Styled Components, etc.]
- **Component Library**: [Storybook, Docusaurus, etc.]
- **Build Tools**: [Vite, Webpack, etc.]

### File Structure
```
src/
├── components/
│   ├── ui/
│   │   ├── Button/
│   │   │   ├── Button.jsx
│   │   │   ├── Button.test.jsx
│   │   │   └── Button.stories.jsx
│   │   └── Input/
│   └── layout/
├── styles/
│   ├── globals.css
│   └── components.css
└── tokens/
    ├── colors.css
    ├── typography.css
    └── spacing.css
```

### Naming Conventions
- **Components**: PascalCase
- **CSS Variables**: kebab-case
- **Utilities**: BEM methodology

## 11. Testing Guidelines

### Visual Testing
- **Screenshots**: Para regresiones visuales
- **Responsive**: Test en todos los breakpoints
- **Cross-browser**: Chrome, Firefox, Safari, Edge

### Accessibility Testing
- **Automated**: axe-core, lighthouse
- **Manual**: Keyboard navigation, screen readers
- **User Testing**: Con usuarios reales

## 12. Maintenance & Governance

### Version Control
- **Semantic Versioning**: Major.Minor.Patch
- **Changelog**: Documentar todos los cambios
- **Backward Compatibility**: Mantener cuando posible

### Review Process
- **Design Review**: Antes de implementar
- **Code Review**: Verificar implementación
- **User Testing**: Validar con usuarios

### Documentation Updates
- **Component Docs**: Mantener actualizadas
- **Guidelines**: Actualizar con cambios
- **Examples**: Reales y funcionales

## 13. Tools & Resources

### Design Tools
- **Figma**: [Link a library]
- **Sketch**: [Link a library]
- **Adobe XD**: [Link a library]

### Development Tools
- **Storybook**: [Link a Storybook]
- **Chromatic**: Para visual testing
- **Lighthouse**: Para performance y accessibility

### Resources
- **Design Tokens**: [Link a tokens]
- **Icon Library**: [Link a icons]
- **Font Files**: [Link a fonts]

---

## Component Index

| Component | Status | Owner | Last Updated |
|---|---|---|---|
| Button | ✅ Implemented | [Name] | [Date] |
| Input | ✅ Implemented | [Name] | [Date] |
| Modal | 🔄 In Progress | [Name] | [Date] |
| Table | 📋 Planned | [Name] | [Date] |

---

## Approval

**Design Lead**: ___________________ Date: ________

**Engineering Lead**: ___________________ Date: ________

**Product Lead**: ___________________ Date: ________
