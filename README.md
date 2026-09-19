# Recipe Web App — ASP.NET Web Forms + SOAP Web Service

A recipe management web application built with ASP.NET Web Forms (.NET Framework 4.7.2), a custom ASMX SOAP Web Service, and a Microsoft Access (.mdb) database. Users can register, log in, add recipes, search by ID, view recipe details, and submit ratings.

This project was originally completed during studies at **Eduvos** (Higher Certificate in Information Systems, Software Development). This version is a refactored release correcting security and functional bugs from the original submission.

---

## Features

- User registration with security-question based account recovery
- Login and session management via query string
- Add, view, and remove recipes
- Recipe rating (1–5 stars)
- SOAP Web Service architecture separating business logic from presentation

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | ASP.NET Web Forms (.NET 4.7.2) |
| Web Service | ASMX SOAP (System.Web.Services) |
| Database | Microsoft Access (.mdb) via OleDb |
| Front-end | HTML, Bootstrap 3, jQuery 3.4.1 |
| IDE | Visual Studio 2019+ |

---

## Project Structure

```
AdvancedCProject/
├── App_Data/
│   └── RecipeDB.mdb          ← Access database (place here)
├── App_Start/
│   ├── BundleConfig.cs
│   ├── FilterConfig.cs
│   └── RouteConfig.cs
├── Controllers/
│   └── HomeController.cs     ← MVC default (unused; Web Forms pages handle routing)
├── Views/                    ← MVC default views (not actively used)
├── Login.aspx / .cs          ← Login page
├── Register.aspx / .cs       ← Registration page
├── MainPage.aspx / .cs       ← Recipe dashboard
├── Recipe.aspx / .cs         ← Single recipe view + rating
├── ForgotPassword.aspx / .cs ← Password reset
├── WebService1.asmx / .cs    ← All database operations exposed as SOAP methods
├── Web.config                ← App config and connection string
├── database_setup.sql        ← SQL Server equivalent schema (for migration)
└── .gitignore
```

---

## Setup Instructions

### Requirements

- Windows machine with Visual Studio 2019 or later
- .NET Framework 4.7.2
- Microsoft Access Database Engine 2010 (for OleDb / .mdb support)
  - Download: https://www.microsoft.com/en-us/download/details.aspx?id=13255
  - If you are on a 64-bit machine, install the 32-bit engine and set your project to build as x86

### Steps

1. Clone the repo:
   ```bash
   git clone https://github.com/AgCat80/csharp-recipe-webapp.git
   cd csharp-recipe-webapp
   ```

2. Place the database file:
   - Copy `RecipeDB.mdb` into the `App_Data/` folder
   - The connection string uses `|DataDirectory|` which resolves to `App_Data/` automatically — no path editing needed

3. Open in Visual Studio:
   - Open `AdvancedCProject.sln`
   - Visual Studio will restore NuGet packages automatically on first build

4. Update the service endpoint:
   - In `Web.config`, find the `<endpoint address>` under `<system.serviceModel>`
   - Replace `PORT` with the actual port Visual Studio assigns to the project (visible in the browser URL when you first run it)

5. Run the project:
   - Press `F5` or click the green Run button
   - The app opens at `https://localhost:PORT/Login.aspx`

### Alternative: SQL Server

A full SQL Server schema is provided in `database_setup.sql`. To migrate:

1. Run `database_setup.sql` in SSMS or Azure Data Studio
2. Update `Web.config` connection string to a SQL Server connection string:
   ```xml
   <add name="RecipeDBConnectionString"
        connectionString="Server=localhost;Database=RecipeDB;Trusted_Connection=True;"
        providerName="System.Data.SqlClient" />
   ```
3. In `WebService1.asmx.cs`, replace `OleDbConnection` with `SqlConnection` and `OleDbCommand` with `SqlCommand`

---

## Bugs Fixed in This Version

| File | Bug | Fix |
|------|-----|-----|
| `WebService1.asmx.cs` | Hardcoded path `C:/Users/Aydon/Documents/RecipeDB.mdb` | Reads from Web.config via `|DataDirectory|` |
| `WebService1.asmx.cs` | SQL injection in every method | All queries use parameterised `OleDbCommand` |
| `WebService1.asmx.cs` | `conn` as instance field (thread safety) | Connection created locally and disposed with `using` |
| `WebService1.asmx.cs` | `addRecipe`: format string ignores recipeName, inserts wrong values | Rewritten with correct columns and parameters |
| `WebService1.asmx.cs` | `rateRecipe`: `{0}` used twice instead of `{0}` and `{1}` | Fixed to use two distinct parameters |
| `WebService1.asmx.cs` | `getRecipeBody`: `WHERE recipeBody = recipeNum` | Fixed to `WHERE recipeID = recipeNum` |
| `WebService1.asmx.cs` | `disconnectFromDB()` called inside try before `finally conn.Close()` | Replaced with `using` blocks |
| `Login.aspx.cs` | Called `UserValidation(email, security)` with username + password | Changed to `validateProfile(username, password)` |
| `Register.aspx.cs` | Triple duplicate using statements | Cleaned to single import set |
| `Register.aspx.cs` | `&` (bitwise AND) used for field validation instead of `&&` | Fixed to `&&` |
| `ForgotPassword.aspx.cs` | `newpassword.Text` — Label, not TextBox | Fixed to `newpasswordbox.Text` |
| `MainPage.aspx.cs` | `userValidation()` always returned `true` | Now checks query string `name` parameter |
| `MainPage.aspx` | `add` and `view` buttons had no `OnClick` attribute | Must be wired in markup |
| `Web.config` | Hardcoded local path in connection string | Replaced with `|DataDirectory|` |

---

## Known Limitations

- **Passwords are stored in plaintext.** In a production application, passwords must be hashed (e.g. using BCrypt). This is preserved here to match the original educational scope.
- **Session management uses query strings.** A real application would use ASP.NET Session or authentication cookies.
- **Microsoft Access** is a 32-bit desktop database not designed for concurrent web traffic. For any real deployment, migrate to SQL Server or MySQL using the provided `database_setup.sql`.

---

## Author

**Gulian Ibrahim** — Cape Town, South Africa  
GitHub: [AgCat80](https://github.com/AgCat80)  
LinkedIn: [gulian-ibrahim-313a03261](https://www.linkedin.com/in/gulian-ibrahim-313a03261/)
