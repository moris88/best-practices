# React Best Practices

Guida alle migliori pratiche per lo sviluppo di applicazioni React scalabili, performanti e manutenibili.

---

## 📌 1. Struttura del Progetto

Organizzare correttamente i file e le cartelle migliora la scalabilità.

### **Struttura Consigliata**

```bash
/src
  /components  # Componenti riutilizzabili
  /hooks       # Custom hooks
  /atoms       # State managments (Redux, Zustand, Recoil, Jotai ecc.)
  /pages       # Pagine della web app
  /services    # Chiamate API (fetch data)
  /utils       # Funzioni di utilità
  /assets      # Immagini, icone, font, ecc.
  main.tsx     # Punto di ingresso principale
  App.tsx      # Componente principale
  index.css    # Stili globali
```

📌 **Consiglio:** Evita una cartella `components` troppo grande, suddividila in sotto-cartelle per funzionalità.

---

## 🚀 2. Componenti e Composizione

React si basa sulla composizione dei componenti: evita componenti monolitici e favorisci il riutilizzo.

### **✅ Best Practices**

- **Separa logica e UI** (Container-Presentational pattern)

  ```tsx
  // Container Component (logica)
  const UserProfileContainer = () => {
    const { data: user } = useUser();
    return <UserProfile user={user} />;
  };

  // Presentational Component (UI)
  const UserProfile = ({ user }: { user: User }) => <div>{user?.name}</div>;
  ```

- **Mantieni i componenti piccoli**
- **Usa componenti controllati per form e input**

  ```tsx
  <input value={name} onChange={(e) => setName(e.target.value)} />
  ```

---

## 🔄 3. Stato e Gestione dei Dati

Scegliere la giusta strategia per lo stato aiuta a migliorare la performance.

### **Quando usare cosa?**

| Stato                     | Strumento consigliato     |
| ------------------------- | ------------------------- |
| Stato locale              | `useState`                |
| Stato derivato            | `useMemo` / `useCallback` |
| Stato globale (semplice)  | `Context React`           |
| Stato globale (complesso) | Redux Toolkit, Zustand    |
| Dati da API               | React Query               |

📌 **Consiglio:** Evita il Context per stati frequentemente aggiornati, perché causa re-render inutili.

```tsx
const UserContext = createContext<User | null>(null);
const useUser = () => useContext(UserContext);
```

Se hai dati asincroni (es. da API), React Query è spesso la scelta migliore:

```tsx
const { data, error, isLoading } = useQuery(["user", userId], fetchUser);
```

---

## 🎭 4. Hooks e Ottimizzazioni

### **✔️ Regole generali per i Custom Hooks**

- Devono iniziare con `use` (`useFetchData`)
- Devono essere riutilizzabili e modulari

```tsx
const useFetchData = (url: string) => {
  const { data, error, isLoading } = useQuery(["data", url], () =>
    fetch(url).then((res) => res.json())
  );
  return { data, error, isLoading };
};
```

### **📌 Ottimizzazioni: `useMemo` e `useCallback`**

```tsx
const filteredUsers = useMemo(() => users.filter((u) => u.active), [users]);
```

```tsx
const handleClick = useCallback(() => {
  console.log("Clicked");
}, []);
```

---

## 🎨 5. Stili con Tailwind CSS

### **Best Practices**

- Usa le utility class direttamente nei componenti:

  ```tsx
  <button className="bg-blue-500 text-white px-4 py-2 rounded-lg">
    Click me
  </button>
  ```

- Usa `tailwind-merge` o `classnames` per classi dinamiche:

  ```tsx
  import { twMerge } from "tailwind-merge";

  const Button = ({ primary }: { primary?: boolean }) => (
    <button
      className={twMerge("px-4 py-2", primary ? "bg-blue-500" : "bg-gray-500")}
    >
      Click me
    </button>
  );
  ```

- Crea classi riutilizzabili con `@apply`:

  ```css
  .btn {
    @apply px-4 py-2 rounded-lg;
  }
  ```

---

## ⚡ 6. Performance e Lazy Loading

### **Lazy Loading dei Componenti**

```tsx
const LazyComponent = lazy(() => import("./LazyComponent"));

<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>;
```

### **Evitare Re-render inutili**

- Usa `React.memo()`:

  ```tsx
  const UserCard = React.memo(({ user }: { user: User }) => (
    <div>{user.name}</div>
  ));
  ```

- Usa `useRef` per mantenere riferimenti senza re-render:

  ```tsx
  const inputRef = useRef<HTMLInputElement>(null);
  ```

---

## 🛠 7. Configurazione ESLint e Prettier

```bash
npm install --save-dev eslint prettier eslint-config-prettier eslint-plugin-prettier eslint-plugin-react eslint-plugin-react-hooks
npx eslint --init
```

```json
// .prettierrc
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "all"
}
```

```json
// .eslintrc.json
{
  "extends": [
    "plugin:react-hooks/recommended",
    "plugin:react/recommended",
    "plugin:prettier/recommended",
    "plugin:react/jsx-runtime"
  ],
  "plugins": ["react", "prettier"],
  "rules": {
    "prettier/prettier": "error"
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  }
}
```

```json
// package.json
"scripts": {
  "lint": "eslint .",
  "lint:fix": "eslint . --fix"
}
```

```bash
npm run lint
npm run lint:fix
```

### **Configurazione Husky e Lint-Staged**

```bash
npm install husky lint-staged --save-dev
npx husky install
npx husky add .husky/pre-commit "npx lint-staged"
```

```json
// package.json
"husky": {
  "hooks": {
    "pre-commit": "lint-staged"
  }
},
"lint-staged": {
  "*.{js,jsx,ts,tsx}": [
    "eslint --fix",
    "prettier --write"
  ]
}
```

---

## ✅ 8. Testing

Usa Jest e React Testing Library per testare i componenti.

### **Scrivere test per componenti**

```tsx
import { render, screen } from "@testing-library/react";
import UserProfile from "./UserProfile";

test("renders user name", () => {
  render(<UserProfile user={{ name: "John Doe" }} />);
  expect(screen.getByText("John Doe")).toBeInTheDocument();
});
```

📌 **Consiglio:** Evita di testare l'implementazione interna, concentrati sul comportamento visibile.

---

## 🔐 9. Sicurezza e Best Practices

- **Evita attacchi XSS**: sanitizza l'input utente
- **Usa `react-error-boundary`** per gestire errori nei componenti
- **Evita di salvare dati sensibili**

---

## 📚 10. Documentazione

- **Documenta i componenti**: usa Storybook o React Styleguidist
- **Documenta le API**: usa Swagger o Postman
- **Documenta il codice**: usa JSDoc per funzioni e componenti
- **Mantieni un README chiaro e aggiornato**

---

## 🚀 11. Actions Github

- **Usa Actions per CI/CD**
- **Esegui test automatici**
- **Esegui linting e formattazione automatica**

---

## 🎯 Conclusione

Seguendo queste best practices, puoi scrivere applicazioni **scalabili, performanti e manutenibili**.
