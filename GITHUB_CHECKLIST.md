# GitHub upload checklist

- [ ] Rename the repository to `stockpilot-python-api` (or another name you prefer).
- [ ] Review README and add your own screenshots after running Swagger UI.
- [ ] Change the demo `SECRET_KEY` and never commit your real `.env`.
- [ ] Run `pip install -r requirements.txt`, `ruff check .`, and `pytest`.
- [ ] Confirm tests pass in your local environment.
- [ ] Create an empty GitHub repository, then run:
  ```bash
  git init
  git add .
  git commit -m "Build StockPilot inventory and order API"
  git branch -M main
  git remote add origin https://github.com/YOUR-USERNAME/stockpilot-python-api.git
  git push -u origin main
  ```
- [ ] Replace `YOUR-USERNAME` with your GitHub username.
