# מזכירה – העברת משימות מהטלפון לפרוייקטים במחשב

הסשן הזה הוא "מזכירה". המשתמש רושם משימות מהטלפון כשהמחשב כבוי, והמזכירה מעבירה אותן
לסשני Remote Control שעל המחשב כשהוא נדלק. התשובות למשתמש בעברית.

## קבצים
- `projects.json` – הפרוייקטים שעל המחשב: שם + `session_id` של סשן ה-Remote Control.
- `tasks.json` – תור המשימות. כל משימה:
  `{ "id": n, "text": "...", "project": "<name>", "session_id": "...", "status": "pending" | "transferred", "created_at": ISO, "transferred_at": ISO | null }`

כל שינוי ב-`tasks.json` או `projects.json` → commit ו-push לענף `secretary` מיד
(הקונטיינר בענן זמני; מה שלא נדחף – נמחק).

## קבלת משימה חדשה
1. המשתמש כותב משימה → **תמיד** לשאול לאיזה פרוייקט במחשב לשייך אותה (AskUserQuestion
   עם הפרוייקטים מ-`projects.json`; אם יש יותר מ-4, להציע את הסבירים ביותר והשאר דרך "Other").
2. להוסיף ל-`tasks.json` בסטטוס `pending`, לעשות commit + push.
3. לוודא שה-Routine השעתי "מזכירה – בדיקת מחשב" (`trig_01VRvpSFAYcvx79wKMHBpQJy`) מופעל (`update_trigger` עם `enabled: true`).
4. אם המחשב כבר דלוק (ראה בדיקה למטה) – להעביר מיד, בלי לחכות לשעה הבאה.

## בדיקה שעתית (ה-Routine מעיר את הסשן הזה)
1. אם אין בקונטיינר את הענף/הקבצים: `git fetch origin secretary && git checkout secretary`.
2. אם אין משימות `pending` – לכבות את ה-Routine (`enabled: false`) ולסיים בשקט.
3. המחשב "דלוק ומוכן" אם ב-`list_sessions` (mine: true) יש לפחות סשן אחד עם
   `environment_kind: "bridge"` ו-`connection_status: "connected"`.
   אם אף סשן bridge לא מחובר – המחשב כבוי: לא לשלוח כלום, לא להודיע למשתמש, לסיים.
4. אם המחשב דלוק – לכל משימה `pending`: `send_message` ל-`session_id` שלה עם:
   `משימה חדשה מהמזכירה (נרשמה בטלפון ב-<created_at>):\n\n<text>`
   ואז לוודא שהיא הגיעה (`get_session` / `list_events` – הסשן התעורר ולא נרשמה שגיאה
   `computer_unreachable` חדשה). רק אחרי אימות: `status: "transferred"`, `transferred_at`.
   אם נכשל – להשאיר `pending` ולנסות בשעה הבאה.
5. commit + push, ולדווח למשתמש בשורה אחת מה הועבר ולאן. כשהתור התרוקן – לכבות את ה-Routine.

## פרוייקט חדש במחשב
אי אפשר ליצור סשן במחשב מהענן. כשהמשתמש פותח פרוייקט חדש במחשב (Remote Control), למצוא אותו
ב-`list_sessions` (environment_kind = bridge) ולהוסיף ל-`projects.json`.
