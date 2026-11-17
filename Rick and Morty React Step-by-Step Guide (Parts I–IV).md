# Rick and Morty React: Step-by-Step Guide (Parts I–IV)

Nov 17, 2026 · @eric wu

## Overview: how to use this guide

You build one Rick and Morty app across four assignments, and each part reuses the pattern from one lecture. Every step below follows the slides' rhythm: **write the Cypress test first and watch it fail (red), build the feature, run it again (green).**

| Part | Lecture | You add | Lecture pattern to copy |
| --- | --- | --- | --- |
| I | 11: Router intro | Vite + `react-router`, App layout, `<Outlet />`, HomePage | `router.jsx`, `index: true` child |
| II | 12: Pages, links, errors | About, Characters (API cards), 404, NavBar, Cypress | `path: '*'`, `<Link>`, `as={Link}` |
| III | 13: Router hooks | Character details page, `useParams`, `useNavigate` | `pokemon/:id` becomes `characters/:id` |
| IV | 14: Outlet context | Favorites (max 4), shared through the Outlet, Tailwind + Bootstrap | `useOutletContext` |

**Translating the Pokemon slides to Rick and Morty:**

| Slides (Pokemon app) | Your app (Rick and Morty) |
| --- | --- |
| `PokeSearch` / `PokeContainer` | `CharactersPage` |
| `PokeCard` | a `CharacterCard` component (Bootstrap `Card`) |
| `PokemonDetailsPage` at `pokemon/:id` | `CharacterDetailsPage` at `characters/:id` |
| PokeAPI | [rickandmortyapi.com](https://rickandmortyapi.com/api/character) |
| `NotFoundPage.jsx` | `NotFound.jsx` (the README's filename; use that) |

**React Router 8 import rules (from lecture 11):**

- `npm install react-router` only. Skip `react-router-dom`, `<BrowserRouter>`, `<Switch>`, `useHistory` and `history.push`.
- `RouterProvider` comes from `'react-router/dom'`. Everything else (`createBrowserRouter`, `Outlet`, `Link`, `useParams`, `useNavigate`, `useOutletContext`) comes from `'react-router'`.
- Needs Node 22.22 or newer.

**Where you end up after Part IV:**

```
src/
  main.jsx          renders <RouterProvider router={router} />
  router.jsx        all routes
  App.jsx           NavBar + <Outlet context={...} />, owns the favorites
  pages/            HomePage, AboutPage, CharactersPage,
                    CharacterDetailsPage, FavoriteCharactersPage, NotFound
  components/       NavBar, CharacterCard, BackButton
cypress/e2e/        navigation.cy.js, characters.cy.js, details.cy.js, favorites.cy.js
```

```
/                  App (NavBar, Outlet)
  index            HomePage
  about            AboutPage
  characters       CharactersPage
  characters/:id   CharacterDetailsPage
  favorites        FavoriteCharactersPage
  *                NotFound
```

## Part I: Vite, React Router and the layout (Lecture 11)

**Goal:** one app, one route. The path `/` shows `App`, and `App` shows `HomePage` inside its `<Outlet />`.

**README requirements:** Vite + React; `HomePage.jsx`, `App.jsx`, `router.jsx`; `createBrowserRouter` connected to `App.jsx`; `HomePage` renders an "Attention Getter" at `http://localhost:5173/`.

### Step 1: Create the project

```sh
npm create vite@latest rick-and-morty -- --template react
cd rick-and-morty
npm install
npm install react-router
npm run dev
```

One package only: `react-router` (lecture 11: `react-router-dom` was merged into it in version 8).

### Step 2: Write the first test (red)

Cypress is required in Part II anyway, so install it now and keep the red/green habit from the start.

```sh
npm install -D cypress
npx cypress open
```

In a second terminal (with `npm run dev` still running), choose **E2E Testing**. It creates `cypress.config.js`; set `baseUrl` so `cy.visit('/')` works:

```js
// cypress.config.js
export default defineConfig({
  e2e: { baseUrl: 'http://localhost:5173' },
})
```

Then create `cypress/e2e/home.cy.js`:

```js
describe('Home page', () => {
  it('shows the attention getter at /', () => {
    cy.visit('/')
    cy.contains('Watch Rick and Morty')
  })
})
```

Run it. It is **red**: no router and no page yet.

### Step 3: Make the page component

A page is just a component that lives in `src/pages` (lecture 11).

```jsx
// src/pages/HomePage.jsx
function HomePage() {
  return (
    <section>
      <h2>Watch Rick and Morty!</h2>
      <p>Interdimensional adventures, one very bad grandpa, zero refunds.</p>
    </section>
  )
}

export default HomePage
```

### Step 4: Turn App into the layout

`App` stays on screen for every page. The `<Outlet />` is the hole the current page fills.

```jsx
// src/App.jsx
import { Outlet } from 'react-router'

function App() {
  return (
    <>
      <header>
        <h1>Rick and Morty</h1>
      </header>
      <main>
        <Outlet />
      </main>
    </>
  )
}

export default App
```

### Step 5: Create the router

A route is an object: `path` is the URL, `element` is the JSX, `children` fill the parent's Outlet. `index: true` means "the default child, shown at the parent's own path", so it has no `path`.

```jsx
// src/router.jsx
import { createBrowserRouter } from 'react-router'
import App from './App'
import HomePage from './pages/HomePage'

const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    children: [{ index: true, element: <HomePage /> }],
  },
])

export default router
```

### Step 6: Render the router instead of App

```jsx
// src/main.jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { RouterProvider } from 'react-router/dom'
import router from './router'
import './index.css'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>
)
```

`RouterProvider` comes from `'react-router/dom'`, not `'react-router'`.

### Step 7: Run it (green) and break it on purpose

1. Re-run `home.cy.js`. It is **green**, and `http://localhost:5173/` shows your attention getter.
2. Delete `<Outlet />` from `App.jsx`. The test goes **red** and you see only the heading (lecture 11 calls this "heading only: the router is half wired").
3. Put `<Outlet />` back. Green again.

### Part I checklist

- [ ] Project created with Vite's React template; `npm run dev` works
- [ ] `react-router` installed (not `react-router-dom`)
- [ ] `src/pages/HomePage.jsx` renders an attention getter
- [ ] `src/App.jsx` renders a heading and an `<Outlet />`
- [ ] `src/router.jsx` uses `createBrowserRouter`: `App` at `/`, `HomePage` as the `index: true` child
- [ ] `src/main.jsx` renders `<RouterProvider router={router} />` (imported from `'react-router/dom'`)
- [ ] The attention getter shows at `http://localhost:5173/`
- [ ] Cypress installed, `baseUrl` set, `home.cy.js` is green
- [ ] Pushed to the `Rick-and-Morty-React` repo

## Part II: Pages, links and errors (Lecture 12)

**Goal:** four real pages (Home, About, Characters, 404) and a NavBar that moves between them without reloading. Every page gets a Cypress test that visits its URL.

**README requirements:** `AboutPage.jsx`; `CharactersPage.jsx` with a Bootstrap `Card` per character from the [Rick and Morty API](https://rickandmortyapi.com/) (mind the pagination); routes as `children` of App in `router.jsx`; `NotFound.jsx` plus a `path: '*'` route; `NavBar.jsx`; Cypress tests; Bootstrap or Tailwind.

Keep working in your Part I project.

### Step 1: Install and wire up Bootstrap

```sh
npm install react-bootstrap bootstrap axios
```

Add this to `src/main.jsx`, **above** `import './index.css'`:

```jsx
import 'bootstrap/dist/css/bootstrap.min.css'
```

(Cypress and its `baseUrl` were set up in Part I.)

### Step 2: About page

**Test first (red).** Create `cypress/e2e/navigation.cy.js`:

```js
describe('Navigation', () => {
  it('/about renders the About page', () => {
    cy.visit('/about')
    cy.contains('h2', 'About')
    cy.contains('h1', 'Rick and Morty') // the layout is still there
  })
})
```

**Build.** Use `h2`, because the header already has the `h1`.

```jsx
// src/pages/AboutPage.jsx
import Card from 'react-bootstrap/Card'

function AboutPage() {
  return (
    <Card>
      <Card.Body>
        <Card.Title as="h2">About the show</Card.Title>
        <Card.Text>
          Rick and Morty follows a mad scientist and his grandson across
          infinite dimensions. Write your own summary here: creators,
          premiere year, main cast.
        </Card.Text>
      </Card.Body>
    </Card>
  )
}

export default AboutPage
```

**Route (green).** Child paths have no leading slash:

```jsx
// src/router.jsx  (inside children)
{ path: 'about', element: <AboutPage /> },
```

Optional stretch from the README: fetch some of the About text from the API (for example the total character count in `info.count` from `https://rickandmortyapi.com/api/character`).

### Step 3: The 404 page

**Test first (red):**

```js
it('shows a 404 page for an unknown route', () => {
  cy.visit('/this-page-does-not-exist')
  cy.contains('does not exist')
})
```

**Build.** `Link` is the router's `<a>`: it changes pages without a reload.

```jsx
// src/pages/NotFound.jsx
import { Link } from 'react-router'

function NotFound() {
  return (
    <section>
      <h2>404</h2>
      <p>This page does not exist.</p>
      <Link to="/">Back to home</Link>
    </section>
  )
}

export default NotFound
```

**Route (green).** `path: '*'` catches every address no other route matched:

```jsx
{ path: '*', element: <NotFound /> },
```

Put it last in `children`. Do not confuse it with `errorElement`, which only shows when a page crashes (see Troubleshooting).

### Step 4: NavBar with Link

**Test first (red).** Mark the window, click About, check the mark survived. A full page reload would wipe it.

```js
it('navigates with the navbar without reloading', () => {
  cy.visit('/')
  cy.window().then((win) => { win.__marker = true })
  cy.contains('a', 'About').click()
  cy.location('pathname').should('eq', '/about')
  cy.window().its('__marker').should('eq', true)
})

it('shows the nav links', () => {
  cy.visit('/')
  cy.contains('a', 'Home').should('be.visible')
  cy.contains('a', 'About').should('be.visible')
  cy.contains('a', 'Characters').should('be.visible')
})
```

**Build.** Use React Bootstrap's `Navbar`. `as={Link}` gives Bootstrap's look with the router's no-reload behaviour. A plain `<a href>` would fail the first test, because it reloads the page.

```jsx
// src/components/NavBar/NavBar.jsx
import Container from 'react-bootstrap/Container'
import Nav from 'react-bootstrap/Nav'
import Navbar from 'react-bootstrap/Navbar'
import { Link } from 'react-router'

function NavBar() {
  return (
    <Navbar bg="dark" data-bs-theme="dark">
      <Container>
        <Navbar.Brand as={Link} to="/">Rick and Morty</Navbar.Brand>
        <Nav>
          <Nav.Link as={Link} to="/">Home</Nav.Link>
          <Nav.Link as={Link} to="/about">About</Nav.Link>
          <Nav.Link as={Link} to="/characters">Characters</Nav.Link>
        </Nav>
      </Container>
    </Navbar>
  )
}

export default NavBar
```

```js
// src/components/NavBar/index.js
export { default } from './NavBar'
```

Show it in `App.jsx` on every page, **above `<main>` and outside `<header>`**, so it stays visible and the header keeps only its heading:

```jsx
// src/App.jsx
import { Outlet } from 'react-router'
import NavBar from './components/NavBar'

function App() {
  return (
    <>
      <header>
        <h1>Rick and Morty</h1>
      </header>
      <NavBar />
      <main>
        <Outlet />
      </main>
    </>
  )
}
```

(`Characters` will not match a route until Step 5; the nav tests only need the link to exist.)

### Step 5: Characters page with API cards and pagination

**Test first (red).** Each API page returns 20 characters, and the first is Rick Sanchez:

```js
// cypress/e2e/characters.cy.js
describe('Characters page', () => {
  it('shows a card for each character on the first page', () => {
    cy.visit('/characters')
    cy.get('.card').should('have.length', 20)
    cy.contains('Rick Sanchez')
  })

  it('loads the next page', () => {
    cy.visit('/characters')
    cy.contains('button', 'Next').click()
    cy.contains('Page 2 of')
  })
})
```

**Build the card:**

```jsx
// src/components/CharacterCard/CharacterCard.jsx
import Card from 'react-bootstrap/Card'

function CharacterCard({ character }) {
  return (
    <Card>
      <Card.Img variant="top" src={character.image} alt={character.name} />
      <Card.Body>
        <Card.Title as="h3">{character.name}</Card.Title>
        <Card.Text>
          {character.status} · {character.species} · {character.gender}
        </Card.Text>
      </Card.Body>
    </Card>
  )
}

export default CharacterCard
```

```js
// src/components/CharacterCard/index.js
export { default } from './CharacterCard'
```

**Build the page.** The effect depends on `[page]`, so it loads again when the page changes. The `ignore` flag stops an old answer from overwriting a newer one. `info.next` and `info.prev` are `null` at the ends, which is what disables the buttons.

```jsx
// src/pages/CharactersPage.jsx
import { useEffect, useState } from 'react'
import axios from 'axios'
import Button from 'react-bootstrap/Button'
import Col from 'react-bootstrap/Col'
import Row from 'react-bootstrap/Row'
import CharacterCard from '../components/CharacterCard'

function CharactersPage() {
  const [characters, setCharacters] = useState([])
  const [info, setInfo] = useState(null)
  const [page, setPage] = useState(1)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    let ignore = false
    setLoading(true)
    setError(null)
    axios
      .get(`https://rickandmortyapi.com/api/character?page=${page}`)
      .then((response) => {
        if (!ignore) {
          setCharacters(response.data.results)
          setInfo(response.data.info)
          setLoading(false)
        }
      })
      .catch(() => {
        if (!ignore) {
          setError('Could not load characters')
          setLoading(false)
        }
      })
    return () => {
      ignore = true
    }
  }, [page])

  if (error) return <p>{error}</p>
  if (loading) return <p>Loading...</p>

  return (
    <>
      <h2>Characters</h2>
      <Row xs={1} md={3} lg={4} className="g-3">
        {characters.map((character) => (
          <Col key={character.id}>
            <CharacterCard character={character} />
          </Col>
        ))}
      </Row>
      <div className="d-flex align-items-center gap-3 my-3">
        <Button disabled={!info.prev} onClick={() => setPage(page - 1)}>
          Prev
        </Button>
        <span>Page {page} of {info.pages}</span>
        <Button disabled={!info.next} onClick={() => setPage(page + 1)}>
          Next
        </Button>
      </div>
    </>
  )
}

export default CharactersPage
```

Route (green):

```jsx
{ path: 'characters', element: <CharactersPage /> },
```

The README says "a Card for every character". Prev/Next reaches every character, one page at a time. If your instructor wants all of them on one screen, loop over `info.pages` and request each page; that is 42 requests, so ask first.

### Step 6: Optional error page (not in the README)

Lecture 12 also shows `errorElement`, which catches a page that **crashes**, unlike `*`, which catches a **missing** URL. Add it only if you want the extra credit: wrap the pages in a pathless route that has `errorElement: <ErrorPage />`, so a crash replaces just the page and the NavBar stays.

### Part II checklist

- [ ] `react-bootstrap`, `bootstrap`, `axios` installed; Bootstrap CSS imported above `index.css`
- [ ] `navigation.cy.js` has a test for `/about` and it passes
- [ ] `AboutPage.jsx` renders show info (h2 heading, Bootstrap `Card`)
- [ ] `NotFound.jsx` shows a "this page does not exist" message with a link home
- [ ] Catch-all `path: '*'` route is the last child of App
- [ ] `NavBar.jsx` uses `Link` (via `as={Link}`), not `<a href>`
- [ ] NavBar shows on every page, above `<main>`, and the no-reload test passes
- [ ] `CharactersPage.jsx` fetches with axios + `useEffect` and renders a `Card` per character
- [ ] Pagination works: Prev and Next use `info.prev` and `info.next`, and disable at the ends
- [ ] Routes for About, Characters and `*` are `children` of App in `router.jsx`
- [ ] Cypress tests cover About, 404, NavBar (no reload) and Characters, all green
- [ ] Pushed to the `Rick-and-Morty-React-II` repo

## Part III: useParams and useNavigate (Lecture 13)

**Goal:** every character gets its own shareable address (`/characters/1`), a details page that loads that character from the URL, and navigation that code decides (a Back button and Previous/Next character).

**README requirements:** `CharacterDetailsPage.jsx` showing all the information about one character with `useParams`; a dynamic route for it; `useNavigate` used in your program.

Keep working in your Part II project. Import `useParams`, `useNavigate` and `useLocation` from `'react-router'`.

### Step 1: A route with a parameter

**Test first (red).** Create `cypress/e2e/details.cy.js`:

```js
describe('Character details', () => {
  it('reads the id from the URL', () => {
    cy.visit('/characters/1')
    cy.contains('Character #1')
  })
})
```

**Build a placeholder page.** `useParams()` returns an object, and the id is **always a string**.

```jsx
// src/pages/CharacterDetailsPage.jsx
import { useParams } from 'react-router'

function CharacterDetailsPage() {
  const { id } = useParams()
  return <p>Character #{id}</p>
}

export default CharacterDetailsPage
```

**Route (green).** The colon makes `:id` a parameter. Add it as another child of App, above the `*` route:

```jsx
{ path: 'characters/:id', element: <CharacterDetailsPage /> },
```

Type `/characters/7` into your own browser to see it work.

### Step 2: Fetch the character for that id

**Test first (red):**

```js
it('shows the character from the API', () => {
  cy.visit('/characters/1')
  cy.contains('Rick Sanchez')
  cy.contains('Human')
  cy.get('img').should('have.attr', 'src').and('include', '1.jpeg')
})
```

Rick Sanchez is character 1 and never changes, which makes him safe to test against (the lecture does the same with Pikachu, always #25).

**Build.** The effect depends on `[id]`, so it loads when the page opens and again if the id changes. The `ignore` flag stops a stale answer from landing.

```jsx
// src/pages/CharacterDetailsPage.jsx
import { useEffect, useState } from 'react'
import { Link, useParams } from 'react-router'
import axios from 'axios'
import Card from 'react-bootstrap/Card'
import ListGroup from 'react-bootstrap/ListGroup'

function CharacterDetailsPage() {
  const { id } = useParams()
  const [character, setCharacter] = useState(null)
  const [error, setError] = useState(null)

  useEffect(() => {
    let ignore = false
    setCharacter(null)
    setError(null)
    axios
      .get(`https://rickandmortyapi.com/api/character/${id}`)
      .then((response) => {
        if (!ignore) setCharacter(response.data)
      })
      .catch(() => {
        if (!ignore) setError('Character not found')
      })
    return () => {
      ignore = true
    }
  }, [id])

  if (error) {
    return (
      <p>
        {error}. <Link to="/characters">Back to characters</Link>
      </p>
    )
  }
  if (!character) return <p>Loading...</p>

  return (
    <Card style={{ maxWidth: '24rem' }}>
      <Card.Img variant="top" src={character.image} alt={character.name} />
      <Card.Body>
        <Card.Title as="h2">{character.name}</Card.Title>
        <Card.Subtitle className="text-muted">Character #{character.id}</Card.Subtitle>
      </Card.Body>
      <ListGroup variant="flush">
        <ListGroup.Item>Status: {character.status}</ListGroup.Item>
        <ListGroup.Item>Species: {character.species}</ListGroup.Item>
        <ListGroup.Item>Type: {character.type || 'none listed'}</ListGroup.Item>
        <ListGroup.Item>Gender: {character.gender}</ListGroup.Item>
        <ListGroup.Item>Origin: {character.origin.name}</ListGroup.Item>
        <ListGroup.Item>Last known location: {character.location.name}</ListGroup.Item>
        <ListGroup.Item>Episodes: {character.episode.length}</ListGroup.Item>
      </ListGroup>
    </Card>
  )
}

export default CharacterDetailsPage
```

Both tests are now green. "All the information" here means every field the API gives that makes sense to show; add `created` or the episode list if you want more.

### Step 3: A character that does not exist

**Test first (red).** The API answers 404 for `/character/99999`. Without a `catch`, the page spins forever and Cypress fails on the uncaught error.

```js
it('shows a message for an unknown character', () => {
  cy.visit('/characters/99999')
  cy.contains('Character not found')
})
```

The `error` state and the `.catch` in Step 2 already make this **green**. If it is red, check that `if (error)` comes **before** the loading check.

### Step 4: A Details link on every card

**Test first (red):**

```js
it('opens the details page from a card', () => {
  cy.visit('/characters')
  cy.contains('.card', 'Rick Sanchez').contains('a', 'Details').click()
  cy.location('pathname').should('eq', '/characters/1')
})
```

**Build.** A button that is really a `Link` gives Bootstrap's look and the router's behaviour:

```jsx
// src/components/CharacterCard/CharacterCard.jsx  (add inside Card.Body)
import Button from 'react-bootstrap/Button'
import { Link } from 'react-router'

<Button as={Link} to={`/characters/${character.id}`} variant="primary">
  Details
</Button>
```

### Step 5: Link or useNavigate?

The rule from lecture 13: **`Link` by default, `navigate` when code decides.**

| Situation | Use |
| --- | --- |
| The user clicks something to go to a page | `<Link to="...">` |
| Go somewhere after logic (a check passed, a request finished) | `navigate('/...')` |
| Back | `navigate(-1)` |

Links are real `<a>` tags, so right-click, "open in new tab" and screen readers keep working. That is why the Details button above is a `Link`, and `useNavigate` is for the next two steps.

### Step 6: A Back button with useNavigate

**Test first.** The browser's Back already works because Links add to the history, so only the first test is red:

```js
it('goes back with the Back button', () => {
  cy.visit('/characters')
  cy.contains('.card', 'Rick Sanchez').contains('a', 'Details').click()
  cy.contains('button', 'Back').click()
  cy.location('pathname').should('eq', '/characters')
})

it('goes back with the browser Back button', () => {
  cy.visit('/characters')
  cy.contains('.card', 'Rick Sanchez').contains('a', 'Details').click()
  cy.go('back')
  cy.location('pathname').should('eq', '/characters')
})
```

**Build.** `navigate(-1)` is the browser's Back. If someone opened the details page directly, there is no "back" inside the app, so go home instead. React Router gives the first entry in the history the key `'default'`.

```jsx
// src/components/BackButton/BackButton.jsx
import Button from 'react-bootstrap/Button'
import { useLocation, useNavigate } from 'react-router'

function BackButton() {
  const navigate = useNavigate()
  const location = useLocation()

  const handleClick = () => {
    if (location.key === 'default') navigate('/')
    else navigate(-1)
  }

  return (
    <Button variant="outline-light" onClick={handleClick}>
      Back
    </Button>
  )
}

export default BackButton
```

```js
// src/components/BackButton/index.js
export { default } from './BackButton'
```

Put it in the NavBar, inside the `<Container>` and outside `<header>`:

```jsx
// src/components/NavBar/NavBar.jsx  (inside Container, after Nav)
import BackButton from '../BackButton'

<BackButton />
```

### Step 7: Previous and Next character (navigate after logic)

The README asks for `useNavigate` "through your program", so use it for one more feature where code decides the destination.

**Test first (red):**

```js
it('moves to the next character', () => {
  cy.visit('/characters/1')
  cy.contains('button', 'Next character').click()
  cy.location('pathname').should('eq', '/characters/2')
})
```

**Build.** Remember `id` is a string, so convert it before adding:

```jsx
// CharacterDetailsPage.jsx
import { Link, useNavigate, useParams } from 'react-router'
import Button from 'react-bootstrap/Button'

const navigate = useNavigate()          // next to useParams()
const numericId = Number(id)

// below the Card, inside the returned JSX:
<div className="d-flex gap-2 my-3">
  <Button disabled={numericId <= 1} onClick={() => navigate(`/characters/${numericId - 1}`)}>
    Previous character
  </Button>
  <Button onClick={() => navigate(`/characters/${numericId + 1}`)}>
    Next character
  </Button>
</div>
```

Because the effect depends on `[id]`, the page reloads the new character on its own. Check this by clicking Next and watching the card change without a full page reload.

### Part III checklist

- [ ] `details.cy.js` exists and every test in it is green
- [ ] Route `characters/:id` is a child of App, above the `*` route
- [ ] `CharacterDetailsPage.jsx` reads the id with `useParams` (imported from `'react-router'`)
- [ ] The page fetches its own character in an effect with `[id]` and an `ignore` flag
- [ ] The page shows image, name, status, species, type, gender, origin, location and episode count
- [ ] An unknown id (for example `/characters/99999`) shows "Character not found" instead of spinning
- [ ] Every `CharacterCard` has a Details button that is a `Link` to `/characters/:id`
- [ ] `BackButton` uses `useNavigate` and is in the NavBar; opening a details URL directly sends it home
- [ ] `useNavigate` is used for at least one more action (Previous and Next character)
- [ ] Typing `/characters/7` into the address bar opens that character
- [ ] Pushed to the `Rick-and-Morty-React-III` repo

## Part IV: Favorites with Outlet context (Lecture 14)

**Goal:** users can add and remove favorite characters (never more than 4), and the list survives moving between pages. The favorites live in `App`, which never unmounts, and reach pages through `<Outlet context={...} />`.

**README requirements:** `FavoriteCharactersPage.jsx` listing the favorites; a route for it as a child of App; add/remove with a limit of 4; React Bootstrap **and** Tailwind for styling; a Cypress suite covering all of it.

Keep working in your Part III project.

### Step 1: Add Tailwind next to Bootstrap

```sh
npm install tailwindcss @tailwindcss/vite
```

(`react-bootstrap` and `bootstrap` are already installed from Part II.)

1. Add the Tailwind plugin to `vite.config.js`:

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [react(), tailwindcss()],
})
```

2. Make sure `src/main.jsx` imports Bootstrap **above** your own CSS:

```jsx
import 'bootstrap/dist/css/bootstrap.min.css'
import './index.css'
```

3. Make the first line of `src/index.css`:

```css
@import "tailwindcss" important;
```

Without `important`, Bootstrap's CSS overrides your Tailwind classes.

4. Restart `npm run dev`, then check it: add `className="text-3xl font-bold"` to the `<h1>` in `App.jsx`. If the heading gets big and bold, both libraries are working.

### Step 2: Why the favorites must live in App

Leaving a page unmounts it and throws away its state (lecture 14). Favorites kept inside `CharactersPage` would vanish when you visit another page. `App` is the layout and never unmounts, so shared state lives there, and the Outlet carries it down. Same rule as lecture 14's "where should this state live?": who needs it, and must it survive a page change?

| State | Where it lives |
| --- | --- |
| The favorites list | `App` (shared through Outlet context) |
| The Characters page number | `CharactersPage` (fine to lose) |
| The details page's character | `CharacterDetailsPage` (comes from the URL) |

### Step 3: Add and show favorites

**Test first (red).** Create `cypress/e2e/favorites.cy.js`:

```js
describe('Favorites', () => {
  it('adds a favorite and shows it on the Favorites page', () => {
    cy.visit('/characters')
    cy.contains('.card', 'Rick Sanchez').contains('button', 'Add favorite').click()
    cy.contains('a', 'Favorites').click()
    cy.location('pathname').should('eq', '/favorites')
    cy.contains('.card', 'Rick Sanchez')
  })
})
```

**Build the state in App.** One toggle function handles both add and remove, and refuses to go past 4:

```jsx
// src/App.jsx
import { useState } from 'react'
import { Outlet } from 'react-router'
import NavBar from './components/NavBar'

const MAX_FAVORITES = 4

function App() {
  const [favorites, setFavorites] = useState([])

  const toggleFavorite = (character) => {
    const alreadyFavorite = favorites.some((c) => c.id === character.id)
    if (alreadyFavorite) {
      setFavorites(favorites.filter((c) => c.id !== character.id))
    } else if (favorites.length < MAX_FAVORITES) {
      setFavorites([...favorites, character])
    }
  }

  return (
    <>
      <header>
        <h1 className="text-3xl font-bold">Rick and Morty</h1>
      </header>
      <NavBar />
      <main className="mx-auto max-w-6xl p-4">
        <Outlet context={{ favorites, toggleFavorite, maxFavorites: MAX_FAVORITES }} />
      </main>
    </>
  )
}

export default App
```

Pages are not written as tags inside `App`, so you cannot pass props to them. The Outlet's `context` is how the layout hands state to whichever page is showing.

**Read it in the Characters page.** Call the hook at the top with your other hooks (before any early `return`), then pass plain props down to the cards:

```jsx
// src/pages/CharactersPage.jsx
import { useOutletContext } from 'react-router'

const { favorites, toggleFavorite, maxFavorites } = useOutletContext()

// in the map:
<CharacterCard
  character={character}
  isFavorite={favorites.some((c) => c.id === character.id)}
  favoritesFull={favorites.length >= maxFavorites}
  onToggleFavorite={toggleFavorite}
/>
```

**Update the card.** Disable the button when the list is full and this character is not already in it:

```jsx
// src/components/CharacterCard/CharacterCard.jsx
function CharacterCard({ character, isFavorite, favoritesFull, onToggleFavorite }) {
  // ...existing Card, plus this inside Card.Body next to the Details button:
  <Button
    variant={isFavorite ? 'outline-danger' : 'outline-primary'}
    disabled={!isFavorite && favoritesFull}
    onClick={() => onToggleFavorite(character)}
  >
    {isFavorite ? 'Remove favorite' : 'Add favorite'}
  </Button>
}
```

**The Favorites page** reads the same context:

```jsx
// src/pages/FavoriteCharactersPage.jsx
import { useOutletContext } from 'react-router'
import Col from 'react-bootstrap/Col'
import Row from 'react-bootstrap/Row'
import CharacterCard from '../components/CharacterCard'

function FavoriteCharactersPage() {
  const { favorites, toggleFavorite, maxFavorites } = useOutletContext()

  return (
    <>
      <h2 className="text-2xl font-semibold mb-3">
        Favorites ({favorites.length}/{maxFavorites})
      </h2>
      {favorites.length === 0 ? (
        <p>No favorites yet. Pick some on the Characters page.</p>
      ) : (
        <Row xs={1} md={2} lg={4} className="g-3">
          {favorites.map((character) => (
            <Col key={character.id}>
              <CharacterCard
                character={character}
                isFavorite
                favoritesFull={favorites.length >= maxFavorites}
                onToggleFavorite={toggleFavorite}
              />
            </Col>
          ))}
        </Row>
      )}
    </>
  )
}

export default FavoriteCharactersPage
```

**Route and link (green).** Add the route above `*`, and a link in the NavBar:

```jsx
// src/router.jsx
{ path: 'favorites', element: <FavoriteCharactersPage /> },

// src/components/NavBar/NavBar.jsx, inside <Nav>
<Nav.Link as={Link} to="/favorites">Favorites</Nav.Link>
```

`useOutletContext` returns whatever object you gave `context`, so keep the **same names on both sides**. A renamed key, an array instead of an object, or using the hook in `App` itself are the classic mistakes (see Troubleshooting).

### Step 4: Remove a favorite

**Test first (red):**

```js
it('removes a favorite', () => {
  cy.visit('/characters')
  cy.contains('.card', 'Rick Sanchez').contains('button', 'Add favorite').click()
  cy.contains('a', 'Favorites').click()
  cy.contains('.card', 'Rick Sanchez').contains('button', 'Remove favorite').click()
  cy.contains('No favorites yet')
})
```

The toggle function in Step 3 already handles removal, so this should go **green** with no new code. If it does not, check that the Favorites page passes `isFavorite` to its cards.

### Step 5: The limit of 4

**Test first (red if the limit is missing):**

```js
it('does not allow more than 4 favorites', () => {
  cy.visit('/characters')
  cy.get('.card').each(($card, index) => {
    if (index < 4) cy.wrap($card).contains('button', 'Add favorite').click()
  })
  cy.get('.card').eq(4).contains('button', 'Add favorite').should('be.disabled')
  cy.contains('a', 'Favorites').click()
  cy.get('.card').should('have.length', 4)
})
```

The limit is enforced twice on purpose: the button is disabled (what the user sees) and `toggleFavorite` refuses a fifth (so the rule holds even if the UI changes).

### Step 6: Favorites survive a change of page

This is the exact bug lecture 14 fixes, so test it explicitly:

```js
it('keeps favorites after visiting another page', () => {
  cy.visit('/characters')
  cy.contains('.card', 'Rick Sanchez').contains('button', 'Add favorite').click()
  cy.contains('a', 'About').click()
  cy.contains('a', 'Characters').click()
  cy.contains('.card', 'Rick Sanchez').contains('button', 'Remove favorite')
})
```

If this is red, the favorites are still held in a page's own state instead of in `App`.

### Step 7: Style with Bootstrap and Tailwind together

The README wants both, so give each a job and stay consistent:

| Library | Use it for |
| --- | --- |
| React Bootstrap | Components: `Navbar`, `Card`, `Button`, `Row`/`Col`, `ListGroup` |
| Tailwind | Spacing, typography, colors and layout around them: `mx-auto max-w-6xl p-4`, `text-2xl font-semibold`, `gap-3` |

Optional style check (adjust it to the classes you used):

```js
it('applies Tailwind styles', () => {
  cy.visit('/')
  cy.get('h1').should('have.css', 'font-weight', '700')
})
```

Optional extra: show the favorite count in the NavBar. `NavBar` is rendered directly by `App`, so it gets a normal prop (`<NavBar favoriteCount={favorites.length} />`); the Outlet context only reaches pages.

### Step 8: Run the whole suite

```sh
npx cypress run
```

All specs (`home`, `navigation`, `characters`, `details`, `favorites`) must pass together before you submit.

### Part IV checklist

- [ ] Tailwind and Bootstrap both work (`@import "tailwindcss" important;` first in `index.css`, Bootstrap CSS imported above it in `main.jsx`)
- [ ] Favorites state lives in `App` and is shared with `<Outlet context={...} />`
- [ ] Pages read it with `useOutletContext()` (same key names as in `App`)
- [ ] Characters can be added and removed from favorites with one toggle button
- [ ] A fifth favorite is impossible: the button is disabled and `toggleFavorite` refuses
- [ ] `FavoriteCharactersPage.jsx` lists the favorites, with an empty-state message
- [ ] `favorites` route is a child of App, and the NavBar has a Favorites link
- [ ] Favorites survive leaving and returning to a page
- [ ] Cypress covers add, show, remove, the limit of 4, and survival across pages
- [ ] `npx cypress run` is fully green
- [ ] Pushed to the `Rick-and-Morty-React-IV` repo

## Troubleshooting and final submission

**Only the heading shows (the Outlet is empty).** App draws its heading as the layout, but the page has nowhere to go. All four causes look identical on screen, so go down the list (lecture 11):

| What is wrong | Why | Fix |
| --- | --- | --- |
| `main.jsx` renders `<App />` | The router never runs | Render `<RouterProvider router={router} />` |
| `App` has no `<Outlet />` | The page has nowhere to go | Add `<Outlet />` where the page goes |
| No index route | At `/` there is no child | `{ index: true, element: <HomePage /> }` |
| A typo in a key (`chidren`) | Unknown keys are ignored, with no error | Spell it `children` |

**Other problems by symptom:**

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `Cannot find module 'react-router-dom'` or an old tutorial's code fails | Wrong package or old API | `npm install react-router`; use `createBrowserRouter`, `useNavigate`, never `useHistory` |
| `RouterProvider` is undefined | Imported from the wrong place | `import { RouterProvider } from 'react-router/dom'` |
| `/about` works from the NavBar but 404s after refresh | Typed address not served | Use the Vite dev server (`npm run dev`), not a plain file server |
| A child route never matches | Leading slash in a child path | `path: 'about'`, not `'/about'` |
| Every unknown URL shows the wrong page, or `*` swallows real pages | `*` placed before other routes, or wrong element | Keep `{ path: '*' }` last |
| NavBar link reloads the page and state resets | `<a href>` or `Nav.Link href` | `<Nav.Link as={Link} to="...">` |
| Only 20 characters appear | The API paginates | Use `info.next`, `info.prev` or the `page` query parameter |
| Details page shows the wrong or stale character | Effect missing `[id]` | `useEffect(..., [id])` with the `ignore` flag |
| `id + 1` gives `"11"` | `useParams` returns strings | `Number(id) + 1` |
| Details page spins forever on a bad id | No `.catch` | Set an `error` state and render it before the loading check |
| Tailwind classes do nothing | Bootstrap overrides them | `@import "tailwindcss" important;` first in `index.css`, and restart the dev server |
| `useOutletContext()` is `null` or fails | Hook used in `App`, or the component is not under the Outlet | Call it in a page (or a component rendered by one); `App` provides the context |
| Favorites vanish when you change page | State is in a page, not in `App` | Move `useState` to `App` and pass it through `<Outlet context={...} />` |
| Destructured favorites are `undefined` | Key names differ between `App` and the page | Use the same names on both sides, and an object (not an array) |

**404 page versus error page.** `path: '*'` shows when no route matches the URL. `errorElement` shows when a page **crashes** (throws while rendering or loading). A missing page is not a broken page, and the README's catch-all is the `*` route.

**Debugging habit from the slides:** open the Console and the address bar first, read what the failing Cypress test said, then fix the one thing it names. Run the test again before you change anything else.

### Final submission checklist

- [ ] Part I pushed to `Rick-and-Morty-React`: router, layout, HomePage
- [ ] Part II pushed to `Rick-and-Morty-React-II`: About, Characters (paginated), 404, NavBar, Cypress tests
- [ ] Part III pushed to `Rick-and-Morty-React-III`: details page, dynamic route, `useParams`, `useNavigate`
- [ ] Part IV pushed to `Rick-and-Morty-React-IV`: favorites (max 4), Outlet context, Bootstrap + Tailwind, full Cypress suite
- [ ] Each repo contains the project at that stage, not just the README
- [ ] `node_modules` is not committed (check `.gitignore`)
- [ ] `npm install && npm run dev` works from a fresh clone
- [ ] `npx cypress run` is green on the final version
- [ ] Every route works when typed directly into the address bar
- [ ] No `react-router-dom`, `<BrowserRouter>`, `useHistory` or plain `<a href>` for internal links anywhere in `src`
