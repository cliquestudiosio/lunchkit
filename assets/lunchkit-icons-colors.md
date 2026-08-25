# Lunchkit Icons & Color Reference

This reference was extracted from `artifacts/web/src` on August 25, 2026.

## Icons

These icons are imported from [`lucide-react`](https://lucide.dev/icons/).

### Navigation, layout, and actions

```text
ArrowLeft
Bell
CalendarDays
CalendarX
ChevronDown
ChevronUp
ExternalLink
Home
LogIn
LogOut
MoreHorizontal
PanelLeftIcon
Pencil
Plus
RefreshCw
RotateCcw
Save
Search
Send
Settings
Trash2
X
```

### People, roles, and organizations

```text
Briefcase
Building
Building2
ChefHat
ClipboardCheck
Mail
ShieldAlert
ShieldCheck
Store
TerminalSquare
User
UserCircle
UserPlus
Users
```

### Meals, orders, and pickup

```text
Camera
DollarSign
Layers3
Leaf
List
Package
PackageCheck
PackageX
QrCode
TrendingUp
Utensils
UtensilsCrossed
Wallet
```

### Feedback, notifications, and messaging

```text
Bug
Info
Lightbulb
MessageSquare
Sparkles
Ticket
Wrench
```

### Status, loading, and system feedback

```text
AlertCircle
AlertTriangle
Archive
Check
CheckCheck
CheckCircle2
Circle
CircleDot
Clock
Clock3
Copy
Loader2
Rocket
XCircle
Zap
```

## Literal hex colors

These are the distinct literal hex values currently used in the web app:

```text
#154527
#173b27
#1f3a26
#1f5c36
#20241f
#244a34
#246133
#2d7d4d
#30663d
#354534
#356c41
#364537
#365445
#3e7c4a
#465045
#495647
#4d5a4c
#526452
#586656
#5c625b
#5d7666
#667062
#687067
#697267
#6b7068
#6b7469
#747a73
#77a789
#8e2920
#a05116
#a6382d
#a63838
#bdc8ba
#c7d3c7
#cfe0d4
#d1d8ce
#d2d9d0
#d8ddd4
#dbe2d7
#dce3d8
#e1e5dc
#e4f3e5
#e5e2da
#e5e9e2
#e6b4ac
#e7f2eb
#e9eee7
#edf3e9
#f3f7f1
#f4f7f0
#f4f8f1
#f5f5f1
#f7fbf8
#fbfcf9
#fff0db
#fff3f1
#ffffff
```

## Literal `rgba()` values

```text
rgba(0, 0, 0, 0.03)
rgba(0, 0, 0, 0.05)
rgba(0, 0, 0, 0.08)
rgba(0, 0, 0, 0.1)
rgba(0,0,0,0.02)
rgba(0,0,0,0.03)
rgba(0,0,0,0.04)
rgba(0,0,0,0.05)
rgba(0,0,0,0.06)
rgba(0,0,0,0.08)
rgba(0,0,0,0.1)
rgba(0,0,0,0.12)
rgba(0,0,0,0.2)
rgba(0,0,0,0.3)
rgba(0,0,0,0.4)
rgba(0,0,0,0.5)
rgba(0,0,0,0.6)
rgba(0,0,0,0.8)
rgba(0,0,0,0.9)
rgba(0,0,0,1)
rgba(17, 28, 18, 0.34)
rgba(17, 28, 18, 0.62)
rgba(255, 255, 255, 0.04)
rgba(255, 255, 255, 0.05)
rgba(255, 255, 255, 0.09)
rgba(255, 255, 255, 0.1)
rgba(31, 58, 38, 0.05)
rgba(45, 125, 77, 0.18)
rgba(62, 124, 74, 0.15)
```

## HSL theme tokens

The primary design system is tokenized with HSL CSS variables in `src/index.css`.
Copy these variables into a project that uses `hsl(var(--token))` utilities.

### Light theme

```css
:root {
  --background: 40 33% 98%;
  --foreground: 20 20% 15%;
  --border: 30 15% 85%;
  --card: 0 0% 100%;
  --card-foreground: 20 20% 15%;
  --card-border: 30 15% 85%;

  --sidebar: 151 48% 28%;
  --sidebar-foreground: 0 0% 100%;
  --sidebar-border: 151 48% 24%;
  --sidebar-primary: 151 48% 24%;
  --sidebar-primary-foreground: 0 0% 100%;
  --sidebar-accent: 151 48% 35%;
  --sidebar-accent-foreground: 0 0% 100%;
  --sidebar-ring: 151 48% 28%;

  --popover: 0 0% 100%;
  --popover-foreground: 20 20% 15%;
  --popover-border: 30 15% 85%;
  --primary: 151 48% 28%;
  --primary-foreground: 0 0% 100%;
  --secondary: 40 25% 94%;
  --secondary-foreground: 20 20% 15%;
  --muted: 35 15% 93%;
  --muted-foreground: 20 10% 45%;
  --accent: 146 39% 88%;
  --accent-foreground: 151 48% 20%;
  --destructive: 0 84% 60%;
  --destructive-foreground: 0 0% 100%;
  --input: 30 15% 85%;
  --ring: 151 48% 28%;

  --chart-1: 148 59% 26%;
  --chart-2: 27 88% 67%;
  --chart-3: 39 18% 93%;
  --chart-4: 240 5% 45%;
  --chart-5: 0 84% 60%;
}
```

### Dark theme

```css
.dark {
  --background: 20 20% 8%;
  --foreground: 40 33% 98%;
  --border: 20 15% 20%;
  --card: 20 20% 12%;
  --card-foreground: 40 33% 98%;
  --card-border: 20 15% 20%;

  --sidebar: 20 20% 8%;
  --sidebar-foreground: 40 33% 98%;
  --sidebar-border: 20 15% 20%;
  --sidebar-primary: 151 48% 58%;
  --sidebar-primary-foreground: 20 20% 8%;
  --sidebar-accent: 20 20% 16%;
  --sidebar-accent-foreground: 40 33% 98%;
  --sidebar-ring: 151 48% 58%;

  --popover: 20 20% 12%;
  --popover-foreground: 40 33% 98%;
  --popover-border: 20 15% 20%;
  --primary: 151 48% 58%;
  --primary-foreground: 20 20% 8%;
  --secondary: 20 20% 16%;
  --secondary-foreground: 40 33% 98%;
  --muted: 20 20% 16%;
  --muted-foreground: 30 10% 70%;
  --accent: 151 30% 22%;
  --accent-foreground: 146 39% 92%;
  --destructive: 0 62% 30%;
  --destructive-foreground: 0 0% 100%;
  --input: 20 15% 20%;
  --ring: 151 48% 58%;

  --chart-1: 142 69% 58%;
  --chart-2: 27 88% 67%;
  --chart-3: 148 38% 15%;
  --chart-4: 148 20% 70%;
  --chart-5: 0 62% 30%;
}
```

## Typography and shape tokens

```css
--app-font-sans: 'DM Sans', sans-serif;
--app-font-serif: 'Fraunces', serif;
--app-font-mono: Menlo, monospace;
--radius: 0.5rem;
```
