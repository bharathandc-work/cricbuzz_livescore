
from playwright.sync_api import sync_playwright, TimeoutError
import os
import re

# ==========================================
# CONFIGURATION
# ==========================================

URL = "https://www.cricbuzz.com/cricket-match/live-scores/recent-matches"

OUTPUT_DIR = os.path.expanduser(
    "~/Documents/Learn/Day 3/CricbuzzReports"
)

os.makedirs(OUTPUT_DIR, exist_ok=True)

# ==========================================
# MAIN AUTOMATION
# ==========================================

with sync_playwright() as p:

    browser = p.chromium.launch(headless=False)
    page = browser.new_page(
        viewport={"width": 1440, "height": 1000}
    )

    try:

        print("Opening Cricbuzz...")

        page.goto(
            URL,
            wait_until="domcontentloaded",
            timeout=60000
        )

        # Wait for webpage content
        page.wait_for_function(
            "document.body && document.body.innerText.length > 200",
            timeout=30000
        )

        print("Reading recent match results...")

        # Extract webpage text
        page_text = page.locator("body").inner_text()

        lines = [
            line.strip()
            for line in page_text.splitlines()
            if line.strip()
        ]

        # ======================================
        # FIND LATEST COMPLETED MATCH
        # ======================================

        result_pattern = re.compile(
            r'\bwon by\s+\d+\s+(?:runs?|wkts?|wickets?)\b'
            r'|\bMatch tied\b|\bMatch drawn\b',
            re.IGNORECASE
        )

        # Team name + score + overs
        score_pattern = re.compile(
            r'^(.+?)\s+'
            r'(\d{1,3}(?:-\d{1,2}|/\d{1,2})?'
            r'(?:\s*&\s*\d{1,3}(?:-\d{1,2}|/\d{1,2})?)?)'
            r'\s*\((\d+(?:\.\d+)?)\)$'
        )

        latest_result = None

        for i, line in enumerate(lines):

            result_match = result_pattern.search(line)

            if result_match:

                # Find the two score lines immediately
                # preceding the result
                previous_lines = lines[max(0, i - 12):i]

                found_scores = []

                for j, previous_line in enumerate(previous_lines):

                    score_match = score_pattern.match(previous_line)

                    if score_match:
                        found_scores.append({
                            "team": score_match.group(1),
                            "score": score_match.group(2),
                            "overs": score_match.group(3)
                        })

                if len(found_scores) >= 2:

                    latest_result = {
                        "scores": found_scores[-2:],
                        "result": line,
                        "result_index": i,
                        "lines": lines
                    }

                    break

        # ======================================
        # DISPLAY RESULT
        # ======================================

        print("\n========== LATEST MATCH RESULT ==========\n")

        if latest_result:

            scores = latest_result["scores"]

            team1 = scores[0]
            team2 = scores[1]

            print(
                f"Match: {team1['team']} vs {team2['team']}"
            )

            print(
                f"Score: {team1['team']} "
                f"{team1['score']} ({team1['overs']} Ov)"
            )

            print(
                f"       {team2['team']} "
                f"{team2['score']} ({team2['overs']} Ov)"
            )

            print(f"Result: {latest_result['result']}")

            # Match information / venue
            score_start = latest_result["result_index"] - 12

            for line in reversed(
                latest_result["lines"][max(0, score_start):latest_result["result_index"]]
            ):

                if re.search(
                    r'\b(?:match|final|qualifier|eliminator|test|odi|t20)\b',
                    line,
                    re.IGNORECASE
                ):
                    print("Match details:", line)
                    break

        else:

            print("No completed match with two scores found.")

            print("\nDebug: Recent webpage content")

            print("\n".join(lines[:40]))

        # ======================================
        # SAVE SCREENSHOT
        # ======================================

        screenshot_path = os.path.join(
            OUTPUT_DIR,
            "score.png"
        )

        page.screenshot(
            path=screenshot_path,
            full_page=True
        )

        print("\nScreenshot saved: score.png")

        print("Location:", screenshot_path)

    except TimeoutError:

        print("Timed out while loading Cricbuzz.")

    except Exception as e:

        print("Error:", e)

    finally:

        browser.close()
