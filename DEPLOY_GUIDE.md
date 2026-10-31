# Sensor Squad — GitHub Pages Publishing Guide

## What to upload
Upload **index.html** to the ROOT of your existing GitHub repository (beside `app.py`, `requirements.txt`, and `README.md`). Do not upload the ZIP file itself and do not delete any existing project files.

## Publish
1. Open GitHub and sign in.
2. Open your existing capstone repository, the one that includes `app.py` and your CSV outputs.
3. Click **Add file** > **Upload files**.
4. Drag in **index.html** and click **Commit changes** to the `main` branch.
5. In the repository, select **Settings** > **Pages**.
6. Under **Build and deployment**, choose **Deploy from a branch**.
7. Select branch **main** and folder **/(root)**; click **Save**.
8. Once GitHub publishes the page, the actual website link is shown in **Settings > Pages**. Open that exact URL.

## Expected website URL (only if the repository name matches exactly)
https://yojanasubedi.github.io/Capstone-Project--Environmental-Sensor-Quality-Monitoring-and-Anomaly-Detection/

## Edit the website later
Open `index.html` in GitHub > pencil icon > edit > **Commit changes**.
- All styling and content are in the same file.
- Search for `REPOSITORY_URL` at the end to correct the GitHub link if necessary.
- The existing Streamlit dashboard link is included.
- The rainfall SVG in the hero is explicitly labelled illustrative, not real readings.
- No separate images, build tool, or JavaScript dependencies are needed.

## Suggested report appendix caption
Figure A4. Sensor Squad GitHub Pages Project Portfolio Website.

## Final checks
- Visit on a phone and laptop.
- Test Streamlit and GitHub links.
- Confirm actual team contributions before publication.
- Ensure that notebook/training code is accessible to assessors if required, without exposing credentials or restricted raw data.
- Put the published GitHub Pages URL in your README and portfolio appendix.
