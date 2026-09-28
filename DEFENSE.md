DEFENSE — Завдання 1: Домен і модель даних (ER)

Намір і критерії (1–2 речення):
Спроєктувати ER-модель для музичного сервісу, яка підтримує складні зв'язки «багато-до-багатьох» з використанням UUID без створення сполучних таблиць.

Топ-3 розбіжності (spec ↔ артефакт) + коміт-виправлення:
•Ai створив id без позначання їх як Primary Key.
•Ai не додав 'user_id FK' у сутність Playlist.
•ШІ проставив 'datetime' замість 'timestamp'.

Виправлено шляхом уточнення критеріїв прийняття у 'spec.md' (коміт: https://github.com/ranenneyoulikemore/music-app-er/commit/c407ee22728272c27cdd0d8e4b2bcb99119d54d6 та 
https://github.com/ranenneyoulikemore/music-app-er/commit/5a85d43d9dd7b64fe9f0a359f506aaada2296255).

Ключове рішення — які альтернативи зважив і чому обрав цю (ADR):
Замість зв'язку 1:N між Artist та Track обрано зв'язок N:M. Альтернатива 1:N спростила б модель, але унеможливила б підтримку дуетів та співавторства треків.(деталі у adr/0001-artist-track-relation.md)

Перевірка — узгодженість із попередньою моделлю (як перевірив):
Перевірила відповідність Mermaid-коду опису в 'spec.md': кількість сутностей, типізація UUID, маркери PK/FK та зв'язки повністю збігаються.

Здача: (а) посилання на GitHub PR;
https://github.com/ranenneyoulikemore/music-app-er/pull/1

 (б) цей заповнений DEFENSE.
https://docs.google.com/document/d/1O6Wed-o6aGPA-pWjx7QDi8Xe3UNf9HLVB5CAi1KrKXk/edit?usp=sharing