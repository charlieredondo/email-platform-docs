# Design System - Email Platform

## Overview

The Email Platform design system is built on top of **Shadcn/ui** and **TailwindCSS**, providing a consistent, accessible, and modern user interface across all components of the application.

## Design Principles

### 1. **Clarity & Simplicity**
- Clean, uncluttered interfaces
- Clear visual hierarchy
- Intuitive navigation patterns
- Minimal cognitive load

### 2. **Accessibility First**
- WCAG 2.1 AA compliance
- Keyboard navigation support
- Screen reader optimization
- High contrast mode support

### 3. **Responsive Design**
- Mobile-first approach
- Breakpoint consistency
- Fluid typography and spacing
- Touch-friendly interactions

### 4. **Performance**
- Optimized component rendering
- Lazy loading strategies
- Minimal bundle sizes
- Fast loading times

## Color Palette

### Primary Colors
```css
:root {
  /* Primary Brand Colors */
  --primary-50: #eff6ff;
  --primary-100: #dbeafe;
  --primary-200: #bfdbfe;
  --primary-300: #93c5fd;
  --primary-400: #60a5fa;
  --primary-500: #3b82f6;  /* Main brand color */
  --primary-600: #2563eb;
  --primary-700: #1d4ed8;
  --primary-800: #1e40af;
  --primary-900: #1e3a8a;
  --primary-950: #172554;
}
```

### Secondary Colors
```css
:root {
  /* Success Colors */
  --success-50: #f0fdf4;
  --success-500: #22c55e;
  --success-600: #16a34a;
  --success-700: #15803d;

  /* Warning Colors */
  --warning-50: #fffbeb;
  --warning-500: #f59e0b;
  --warning-600: #d97706;
  --warning-700: #b45309;

  /* Error Colors */
  --error-50: #fef2f2;
  --error-500: #ef4444;
  --error-600: #dc2626;
  --error-700: #b91c1c;

  /* Info Colors */
  --info-50: #f0f9ff;
  --info-500: #06b6d4;
  --info-600: #0891b2;
  --info-700: #0e7490;
}
```

### Neutral Colors
```css
:root {
  /* Neutral/Gray Scale */
  --neutral-50: #f8fafc;
  --neutral-100: #f1f5f9;
  --neutral-200: #e2e8f0;
  --neutral-300: #cbd5e1;
  --neutral-400: #94a3b8;
  --neutral-500: #64748b;
  --neutral-600: #475569;
  --neutral-700: #334155;
  --neutral-800: #1e293b;
  --neutral-900: #0f172a;
  --neutral-950: #020617;
}
```

## Typography

### Font Families
```css
:root {
  /* Primary font for UI */
  --font-sans: 'Inter', system-ui, -apple-system, sans-serif;
  
  /* Monospace for code/data */
  --font-mono: 'JetBrains Mono', 'Fira Code', Consolas, monospace;
  
  /* Display font for headings */
  --font-display: 'Inter', system-ui, sans-serif;
}
```

### Type Scale
```css
/* Heading Styles */
.text-display-2xl { font-size: 4.5rem; line-height: 1.1; font-weight: 800; }
.text-display-xl  { font-size: 3.75rem; line-height: 1.1; font-weight: 800; }
.text-display-lg  { font-size: 3rem; line-height: 1.2; font-weight: 700; }
.text-display-md  { font-size: 2.25rem; line-height: 1.2; font-weight: 700; }
.text-display-sm  { font-size: 1.875rem; line-height: 1.3; font-weight: 600; }

/* Body Text Styles */
.text-xl          { font-size: 1.25rem; line-height: 1.7; }
.text-lg          { font-size: 1.125rem; line-height: 1.6; }
.text-base        { font-size: 1rem; line-height: 1.5; }
.text-sm          { font-size: 0.875rem; line-height: 1.4; }
.text-xs          { font-size: 0.75rem; line-height: 1.3; }

/* Code/Data Styles */
.text-code-lg     { font-family: var(--font-mono); font-size: 1rem; }
.text-code        { font-family: var(--font-mono); font-size: 0.875rem; }
.text-code-sm     { font-family: var(--font-mono); font-size: 0.75rem; }
```

## Spacing System

### Base Spacing Scale
```css
/* Based on 4px grid system */
--space-0: 0;
--space-1: 0.25rem;  /* 4px */
--space-2: 0.5rem;   /* 8px */
--space-3: 0.75rem;  /* 12px */
--space-4: 1rem;     /* 16px */
--space-5: 1.25rem;  /* 20px */
--space-6: 1.5rem;   /* 24px */
--space-8: 2rem;     /* 32px */
--space-10: 2.5rem;  /* 40px */
--space-12: 3rem;    /* 48px */
--space-16: 4rem;    /* 64px */
--space-20: 5rem;    /* 80px */
--space-24: 6rem;    /* 96px */
```

### Component Spacing
```css
/* Container padding */
.container-padding { padding: var(--space-4) var(--space-6); }

/* Section spacing */
.section-spacing { margin-bottom: var(--space-12); }

/* Element spacing */
.element-gap { gap: var(--space-4); }
.element-gap-sm { gap: var(--space-2); }
.element-gap-lg { gap: var(--space-6); }
```

## Component Library

### 1. **Button Components**

#### Primary Button
```tsx
<Button variant="default" size="default">
  Send Email
</Button>
```

#### Button Variants
```tsx
// Available variants
<Button variant="default">Primary Action</Button>
<Button variant="secondary">Secondary Action</Button>
<Button variant="outline">Outline Button</Button>
<Button variant="ghost">Ghost Button</Button>
<Button variant="destructive">Delete Action</Button>

// Available sizes
<Button size="sm">Small</Button>
<Button size="default">Default</Button>
<Button size="lg">Large</Button>
```

#### Button States
```css
/* Default state */
.btn-primary {
  background: var(--primary-500);
  color: white;
  border: 1px solid var(--primary-500);
}

/* Hover state */
.btn-primary:hover {
  background: var(--primary-600);
  border-color: var(--primary-600);
}

/* Focus state */
.btn-primary:focus {
  outline: 2px solid var(--primary-500);
  outline-offset: 2px;
}

/* Disabled state */
.btn-primary:disabled {
  background: var(--neutral-300);
  color: var(--neutral-500);
  cursor: not-allowed;
}
```

### 2. **Input Components**

#### Text Input
```tsx
<Input 
  type="text"
  placeholder="Enter email subject"
  value={value}
  onChange={onChange}
/>
```

#### Input with Label
```tsx
<div className="form-field">
  <Label htmlFor="email">Email Address</Label>
  <Input 
    id="email"
    type="email"
    placeholder="user@example.com"
    required
  />
</div>
```

#### Input States
```css
/* Default state */
.input {
  border: 1px solid var(--neutral-300);
  background: white;
  padding: var(--space-3) var(--space-4);
  border-radius: 0.375rem;
}

/* Focus state */
.input:focus {
  border-color: var(--primary-500);
  outline: none;
  box-shadow: 0 0 0 3px var(--primary-100);
}

/* Error state */
.input.error {
  border-color: var(--error-500);
}

/* Disabled state */
.input:disabled {
  background: var(--neutral-100);
  color: var(--neutral-500);
}
```

### 3. **Data Display Components**

#### Table Component
```tsx
<Table>
  <TableHeader>
    <TableRow>
      <TableHead>Email</TableHead>
      <TableHead>Status</TableHead>
      <TableHead>Sent Date</TableHead>
      <TableHead>Actions</TableHead>
    </TableRow>
  </TableHeader>
  <TableBody>
    <TableRow>
      <TableCell>user@example.com</TableCell>
      <TableCell>
        <Badge variant="success">Delivered</Badge>
      </TableCell>
      <TableCell>2025-01-15 10:30</TableCell>
      <TableCell>
        <Button variant="ghost" size="sm">View</Button>
      </TableCell>
    </TableRow>
  </TableBody>
</Table>
```

#### Card Component
```tsx
<Card>
  <CardHeader>
    <CardTitle>Email Analytics</CardTitle>
    <CardDescription>
      Overview of email performance for the last 30 days
    </CardDescription>
  </CardHeader>
  <CardContent>
    <div className="metrics-grid">
      <MetricCard 
        title="Total Sent"
        value="15,420"
        change="+12%"
        trend="up"
      />
    </div>
  </CardContent>
</Card>
```

#### Badge Component
```tsx
// Status badges
<Badge variant="success">Delivered</Badge>
<Badge variant="warning">Queued</Badge>
<Badge variant="error">Failed</Badge>
<Badge variant="info">Processing</Badge>
```

### 4. **Navigation Components**

#### Sidebar Navigation
```tsx
<aside className="sidebar">
  <nav className="sidebar-nav">
    <NavItem href="/dashboard" icon={DashboardIcon}>
      Dashboard
    </NavItem>
    <NavItem href="/emails" icon={EmailIcon}>
      Emails
    </NavItem>
    <NavItem href="/templates" icon={TemplateIcon}>
      Templates
    </NavItem>
    <NavItem href="/analytics" icon={AnalyticsIcon}>
      Analytics
    </NavItem>
  </nav>
</aside>
```

#### Breadcrumb Navigation
```tsx
<Breadcrumb>
  <BreadcrumbItem>
    <BreadcrumbLink href="/dashboard">Dashboard</BreadcrumbLink>
  </BreadcrumbItem>
  <BreadcrumbSeparator />
  <BreadcrumbItem>
    <BreadcrumbLink href="/templates">Templates</BreadcrumbLink>
  </BreadcrumbItem>
  <BreadcrumbSeparator />
  <BreadcrumbItem isCurrentPage>
    <BreadcrumbPage>Edit Template</BreadcrumbPage>
  </BreadcrumbItem>
</Breadcrumb>
```

### 5. **Feedback Components**

#### Toast Notifications
```tsx
// Success toast
toast.success("Email sent successfully", {
  description: "Your email has been queued for delivery",
});

// Error toast
toast.error("Failed to send email", {
  description: "Please check your template configuration",
});

// Info toast
toast.info("Template saved as draft", {
  description: "Publish when ready to use",
});
```

#### Alert Component
```tsx
<Alert variant="destructive">
  <AlertTriangleIcon className="h-4 w-4" />
  <AlertTitle>High Bounce Rate Detected</AlertTitle>
  <AlertDescription>
    Your bounce rate has exceeded 5% in the last hour. 
    Consider reviewing your email list quality.
  </AlertDescription>
</Alert>
```

### 6. **Chart Components**

#### Line Chart (Email Volume)
```tsx
<LineChart
  data={emailVolumeData}
  xAxis={{ key: 'date', label: 'Date' }}
  yAxis={{ key: 'volume', label: 'Emails Sent' }}
  color="var(--primary-500)"
  height={300}
/>
```

#### Bar Chart (Performance Metrics)
```tsx
<BarChart
  data={performanceData}
  xAxis={{ key: 'emailType', label: 'Email Type' }}
  yAxis={{ key: 'deliveryRate', label: 'Delivery Rate %' }}
  color="var(--success-500)"
  height={250}
/>
```

#### Donut Chart (Email Status Distribution)
```tsx
<DonutChart
  data={statusDistribution}
  valueKey="count"
  nameKey="status"
  colors={[
    'var(--success-500)',
    'var(--warning-500)', 
    'var(--error-500)',
    'var(--info-500)'
  ]}
/>
```

## Layout System

### 1. **Dashboard Layout**
```tsx
<div className="dashboard-layout">
  <Sidebar />
  <main className="main-content">
    <Header />
    <div className="content-area">
      <Breadcrumb />
      <PageContent />
    </div>
  </main>
</div>
```

### 2. **Content Grid**
```css
/* Dashboard grid system */
.dashboard-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: var(--space-6);
  padding: var(--space-6);
}

/* Responsive breakpoints */
@media (max-width: 768px) {
  .dashboard-grid {
    grid-template-columns: 1fr;
    padding: var(--space-4);
  }
}
```

### 3. **Form Layouts**
```tsx
<form className="form-layout">
  <div className="form-section">
    <h3 className="form-section-title">Template Details</h3>
    <div className="form-grid">
      <FormField>
        <Label>Template Name</Label>
        <Input type="text" />
      </FormField>
      <FormField>
        <Label>Email Type</Label>
        <Select>
          <SelectItem value="welcome">Welcome</SelectItem>
          <SelectItem value="order">Order Confirmation</SelectItem>
        </Select>
      </FormField>
    </div>
  </div>
</form>
```

## Dark Mode Support

### CSS Variables for Theming
```css
/* Light theme (default) */
:root {
  --background: var(--neutral-50);
  --foreground: var(--neutral-900);
  --card: white;
  --card-foreground: var(--neutral-900);
  --border: var(--neutral-200);
  --input: white;
  --ring: var(--primary-500);
}

/* Dark theme */
[data-theme="dark"] {
  --background: var(--neutral-950);
  --foreground: var(--neutral-50);
  --card: var(--neutral-900);
  --card-foreground: var(--neutral-50);
  --border: var(--neutral-800);
  --input: var(--neutral-900);
  --ring: var(--primary-400);
}
```

### Theme Toggle Component
```tsx
<Button
  variant="ghost"
  size="sm"
  onClick={toggleTheme}
  aria-label="Toggle theme"
>
  {theme === 'dark' ? <SunIcon /> : <MoonIcon />}
</Button>
```

## Responsive Design

### Breakpoint System
```css
/* Mobile first breakpoints */
--screen-sm: 640px;   /* Small devices */
--screen-md: 768px;   /* Medium devices */
--screen-lg: 1024px;  /* Large devices */
--screen-xl: 1280px;  /* Extra large devices */
--screen-2xl: 1536px; /* 2x Extra large devices */
```

### Responsive Patterns
```css
/* Responsive navigation */
@media (max-width: 768px) {
  .sidebar {
    transform: translateX(-100%);
    position: fixed;
    z-index: 50;
  }
  
  .sidebar.open {
    transform: translateX(0);
  }
}

/* Responsive typography */
@media (max-width: 640px) {
  .text-display-xl {
    font-size: 2.5rem;
  }
  
  .text-display-lg {
    font-size: 2rem;
  }
}
```

## Animation Guidelines

### Transition Standards
```css
/* Standard transitions */
.transition-standard {
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}

/* Smooth interactions */
.button {
  transition: background-color 0.2s ease, 
              border-color 0.2s ease,
              box-shadow 0.2s ease;
}

/* Loading animations */
@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.spinner {
  animation: spin 1s linear infinite;
}
```

### Motion Principles
- **Purposeful**: Animations should enhance usability
- **Fast**: Keep animations under 300ms for UI interactions
- **Natural**: Use easing functions that feel natural
- **Accessible**: Respect `prefers-reduced-motion` setting

## Accessibility Guidelines

### ARIA Labels and Roles
```tsx
// Proper labeling
<Button aria-label="Delete email template">
  <TrashIcon aria-hidden="true" />
</Button>

// Form accessibility
<Input
  aria-describedby="email-help"
  aria-invalid={hasError}
  aria-required="true"
/>
<div id="email-help" className="help-text">
  Enter a valid email address
</div>
```

### Keyboard Navigation
```css
/* Focus indicators */
.focusable:focus {
  outline: 2px solid var(--primary-500);
  outline-offset: 2px;
}

/* Skip links */
.skip-link {
  position: absolute;
  top: -40px;
  left: 6px;
  background: var(--primary-500);
  color: white;
  padding: 8px;
  text-decoration: none;
  border-radius: 4px;
}

.skip-link:focus {
  top: 6px;
}
```

## Component Documentation

Each component should include:
1. **Usage examples** with code snippets
2. **Props documentation** with TypeScript interfaces
3. **Accessibility notes** and ARIA patterns
4. **Responsive behavior** specifications
5. **Theme variations** for light/dark modes

This design system ensures consistency, accessibility, and maintainability across the entire Email Platform application while providing a modern and intuitive user experience.