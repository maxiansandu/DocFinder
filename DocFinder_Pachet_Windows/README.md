# Optimizarea infrastructurii IT și a gestionării documentelor

Două livrabile distincte:

1. **Prezentarea HTML**: `Prezentare/Prezentare.html`. Un singur fișier care se deschide direct în browser, cu CSS, JavaScript și capturile de ecran incluse.
2. **Aplicația desktop DocFinder**: un program real pentru căutarea documentelor după nume și conținut, cu cod sursă, executabile, teste și documente demonstrative.

## Conținut

| Cale | Ce este |
|------|---------|
| `Prezentare/Prezentare.html` | Prezentarea pentru director (18 slide-uri, navigare cu `←` `→`, `F` = ecran complet, `L` = mod lectură) |
| `Prezentare/Propunere_IT_si_documente.docx` | Documentul Word pentru director: problemele definite, întrebări cu răspunsuri Da / Nu, soluțiile, pașii și deciziile de bifat |
| `Prezentare/Prezentare_simpla.pptx` | Varianta scurtă PowerPoint (9 slide-uri, pentru colegi fără pregătire IT), cu note pentru prezentator |
| `Prezentare/capturi/` | Capturi de ecran reale ale aplicației (sursa imaginilor din prezentare) |
| `Executabile/DocFinder-Windows-x64/DocFinder.exe` | Aplicația pentru Windows 10/11 x64; nu necesită instalarea .NET |
| `Executabile/DocFinder-Linux-x64/DocFinder` | Aplicația pentru Linux x64 |
| `Porneste-DocFinder.bat`, `porneste-docfinder.sh` | Scurtături de pornire |
| `Documente_Demo/` | 27 de documente **fictive** (PDF, DOCX, TXT etc.) în 20 de foldere, dezorganizate intenționat |
| `Scenariu_demonstratie.md` | Scenariul de ≈ 3 minute, căutările de test și răspunsuri la întrebări |
| `DocFinder/` | Codul sursă (C# / .NET 9 / Avalonia) și testele |
| `tools/` | Scripturi: generarea documentelor demonstrative, testul interfeței, capturile, construirea prezentării |

## Pornire rapidă

**Windows:** dublu clic pe `Porneste-DocFinder.bat` (sau pe `Executabile\DocFinder-Windows-x64\DocFinder.exe`). La prima pornire, Windows SmartScreen poate afișa „Windows protected your PC”, deoarece executabilul nu este semnat digital. Alegeți *More info → Run anyway* doar dacă fișierul provine din această livrare.

**Linux:** `./porneste-docfinder.sh`

În aplicație: **Schimbă folderul → Răsfoiește…** → selectați `Documente_Demo`, apoi scrieți în bara de căutare.

Opțional, din linia de comandă: `DocFinder --source "D:\Documente"` pornește direct cu un folder, iar `--query text` pornește cu o căutare completată.

## Ce face aplicația (implementat și testat)

- Căutare după numele complet sau parțial, cu rezultate afișate în timp ce scrieți.
- Căutare în conținut pentru **PDF cu text selectabil**, **DOCX** și **TXT** (UTF-8 și Windows-1250), cu fragmentul în care apare termenul evidențiat.
- Majusculele și diacriticele nu contează (`neconcurenta` găsește „neconcurență”). Expresiile exacte se scriu între ghilimele. Când scrieți mai mulți termeni, toți trebuie să apară.
- Cuvintele se potrivesc după început: `expir` găsește *expiră, expirare, expirării*.
- Filtre: tipul documentului, data modificării, căutare în nume și/sau în conținut.
- Pentru fiecare rezultat: nume, tip, calea completă, data modificării, dimensiune și fragment. Acțiuni: **Deschide**, **Deschide folderul**, **Copiază calea**.
- Alegerea folderului: local sau cale de rețea (de ex. `\\SERVER\Documente`), plus lista folderelor recente.
- Index local (SQLite FTS5). **Actualizează indexul** (`F5`) procesează doar fișierele noi sau modificate și scoate din index fișierele șterse.
- Mesaje clare pentru: lipsa rezultatelor, foldere fără permisiune, fișiere care nu pot fi citite, PDF-uri scanate. Indexarea afișează progresul și poate fi oprită.
- Tastatură: `Ctrl+F` = căutare, `Enter`/`↓` = primul rezultat, `Enter` = deschide, `Esc` = șterge căutarea, `Ctrl+Shift+C` = copiază calea.

## Ce NU face (declarat deschis)

- **OCR**: PDF-urile scanate se găsesc doar după nume și sunt marcate „necesită OCR”.
- Conținutul altor formate (`.doc` vechi, `.xlsx`, imagini etc.) nu este citit; aceste fișiere se găsesc doar după nume.
- **Căutarea după sens (semantică)** nu este implementată și nu este simulată. Este propusă pentru etapa AI.
- **Nu este conectată la serverul companiei.** Funcționează pe orice folder accesibil utilizatorului; configurarea pe server este etapa următoare, după validare.
- Nu este declarată pregătită pentru producție (vezi „Securitate”).

## Securitate

- **Doar citire:** fișierele sunt deschise cu `FileAccess.Read` și partajare completă, deci nu sunt blocate pentru colegi. Aplicația nu modifică, nu mută și nu șterge nimic. Un test automat compară amprentele SHA-256 și datele tuturor fișierelor înainte și după indexare.
- **Drepturile existente:** aplicația rulează cu contul utilizatorului (manifest `asInvoker`, fără drepturi de administrator). Folderele inaccesibile sunt omise și raportate, nu ocolite. Legăturile simbolice și joncțiunile nu sunt urmate.
- **Fără rețea:** aplicația nu are nicio conexiune la internet și nu folosește servicii externe.
- **Indexul local** se află în `%LOCALAPPDATA%\DocFinder\index` (Linux: `~/.local/share/DocFinder/index`) și **conține text extras din documente**. Trebuie protejat ca documentele, de exemplu prin criptarea discului. Locația se poate schimba din configurare.
- Înainte de producție: test pe calculatoarele Windows ale companiei, în rețea; verificarea permisiunilor pe folderele partajate; o regulă pentru index (locație, criptare, ștergere).

## Configurarea pentru server (etapa următoare)

Fișierul `docfinder.config.json`, aflat lângă executabil, este editat de administrator:

```json
{
  "DefaultSourcePath": "\\\\SERVER\\Documente",
  "ExcludedFolderNames": [ "$RECYCLE.BIN", "System Volume Information" ],
  "MaxFileSizeMB": 50,
  "IndexDirectory": ""
}
```

Fiecare utilizator indexează doar ce are dreptul să citească. Nu este necesară nicio modificare pe server.

## Ce a fost verificat

| Verificare | Rezultat |
|------------|----------|
| 32 de teste automate (`dotnet test`): extragere, index, căutare, toți pașii demonstrației, doar-citire, foldere fără permisiune, anulare, fișiere corupte | toate trec |
| Test pe fereastra reală (Linux, executabilul publicat), cu clicuri și tastare simulate: `tools/linux_ui_smoke_test.py` | 11/11 verificări: alegerea folderului, căutări, filtru, OCR, fără rezultate, copiere cale (citită din clipboard), deschidere document, deschidere folder |
| Executabilul Windows (`DocFinder.exe`) | generat și pornit sub **Wine 11** (mediu de compatibilitate Windows pe Linux): indexează și caută corect. **Nu a fost testat pe un PC Windows real**; acesta este primul pas recomandat. |
| Prezentarea | verificată în browser la 1366×768 (proiector) și 375 px (telefon); navigare, taburi, galerie, plan, decizii |

## Construire din sursă

Necesită [.NET 9 SDK](https://dotnet.microsoft.com/download).

```bash
cd DocFinder
dotnet test
dotnet run --project src/DocFinder.App
```

Executabile autonome (fără .NET instalat):

```bash
dotnet publish DocFinder/src/DocFinder.App -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -p:IncludeNativeLibrariesForSelfExtract=true -p:EnableCompressionInSingleFile=true -p:DebugType=none -o Executabile/DocFinder-Windows-x64
```

Pentru Linux înlocuiți `win-x64` cu `linux-x64`. Pentru macOS (nepublicat și netestat) folosiți `osx-arm64`.

Structura codului:

- `src/DocFinder.Core`: extragerea textului (`Extraction/TextExtractor.cs`), indexul SQLite FTS5 (`Index/DocumentIndex.cs`), indexarea incrementală (`Index/Indexer.cs`), interogările (`Search/QueryParser.cs`), deschiderea fișierelor și folderelor (`ShellActions.cs`), configurarea (`AppConfig.cs`).
- `src/DocFinder.App`: interfața Avalonia (`MainWindow.axaml`, `ViewModels/MainViewModel.cs`).
- `tests/DocFinder.Tests`: testele xUnit.

Biblioteci: Avalonia 11.3, PdfPig 0.1.16 (PDF), Microsoft.Data.Sqlite 9.0 (FTS5). DOCX este citit direct din formatul OpenXML, fără dependențe suplimentare.

## Regenerare (opțional)

```bash
/usr/bin/python3 tools/generate_demo_docs.py                 # documentele fictive (reportlab, PyMuPDF, LibreOffice)
/usr/bin/python3 tools/linux_ui_smoke_test.py Executabile/DocFinder-Linux-x64/DocFinder /tmp/capturi-test
/usr/bin/python3 tools/capture_screenshots.py Executabile/DocFinder-Linux-x64/DocFinder Prezentare/capturi
/usr/bin/python3 tools/build_presentation.py                 # reconstruiește Prezentare/Prezentare.html
NODE_PATH=<node_modules cu pptxgenjs, react-icons, sharp> node tools/build_pptx.js <apply_theme.js>   # Prezentare_simpla.pptx
NODE_PATH=<node_modules cu docx> node tools/build_docx.js                                          # Propunere_IT_si_documente.docx
```

Testul de interfață și capturile mișcă mouse-ul și tastează în fereastra aplicației (X11/XWayland). Nu folosiți calculatorul în acest timp.
