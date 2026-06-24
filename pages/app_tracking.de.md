---
title: Wie Apps nachverfolgt werden
description: Wie Apps in Obtainium nachverfolgt werden
---

# Wie Apps nachverfolgt werden

## Grundlagen

Wenn Sie eine App-URL zu Obtainium hinzufügen, müssen Sie eine [App-Quelle](sources.de.md) auswählen. Quellen definieren, wie App-Informationen und APK-Dateien aus der von Ihnen eingegebenen URL extrahiert werden. In den meisten Fällen wählt Obtainium automatisch die entsprechende Quelle aus – um diese Auswahl zu korrigieren, wird ein Dropdown-Menü „Quelle überschreiben“ angezeigt.

Eine App-Quelle muss mindestens die folgenden Daten für ihre Apps bereitstellen:

- Die App-Version (oder eine „Pseudo-Version“ – eine Kennung, die sich bei jeder neuen Version der App ändert.)
- Mindestens eine APK-Download-URL, die der zur Verfügung gestellten Version entspricht

App-Quellen können auch andere Informationen liefern – diese ermöglichen zusätzliche Funktionen oder UI-Vorteile. Zum Beispiel:

- Der Name des Autors der Anwendung
- Die Package ID der Anwendung
- Das Veröffentlichungsdatum der neuesten Version
- Infos zu früheren Versionen oder Varianten der App

In einer idealen Welt würde jede App-Quelle alle erforderlichen Informationen auf unkomplizierte Weise bereitstellen - mit einer einzigen App pro angegebener URL, die alle erforderlichen Informationen in einem Standardformat enthält. Dies ist jedoch oft nicht der Fall – es gibt viele verschiedene Arten, wie App-Veröffentlichungen gehandhabt werden, sogar von derselben Quelle, so dass es nicht möglich ist, einen festen Satz von Schritten zu haben, um sie alle zu behandeln. Aus diesem Grund werden Ihnen beim Hinzufügen einer App verschiedene zusätzliche Optionen angeboten, mit denen Sie die Art und Weise, wie die App-Informationen extrahiert werden, ändern können. Während die Standardeinstellungen für die meisten Apps funktionieren, sollten Sie diese Optionen kennen, um mit Sonderfällen umgehen zu können – mehr dazu auf der Seite [App-Quellen](sources.de.md).

Hinweis: Viele Filtereinstellungen in Obtainium (einschließlich vieler quellenspezifischer optionaler Filter) verwenden [reguläre Ausdrücke](https://developer.mozilla.org/de/docs/Web/JavaScript/Guide/Regular_expressions) – Sie sollten mit diesen vertraut sein.

## Versionserkennung

Wenn Obtainium eine App trackt, die aktuell installiert ist, holt es sich die Version der App von Android und vergleicht sie mit der von der Quelle bereitgestellten Versionszeichenfolge. Dann vergleicht es die beiden, um zu entscheiden, ob ein Update verfügbar ist oder ob sich der Installationsstatus der App geändert hat. Dieser Vergleich kann nur durchgeführt werden, wenn die beiden Versionen das gleiche Format haben, was nicht immer der Fall ist. Es kann zum Beispiel einer der folgenden Fälle eintreten:

1. [Obtainium](https://github.com/ImranR98/Obtainium/releases/tag/v0.14.21-beta) von GitHub:

    - **Von Android gemeldete App-Version:** `0.14.21`
    - **Von der Quelle gemeldete Version:** `v0.14.21-beta` 

2. [Cheogram](https://git.singpolyma.net/cheogram-android/refs/2.12.8-2) von einer SourceHut-Instanz:

    - **Von Android gemeldete App-Version:** `2.12.8-2+free`
    - **Von der Quelle gemeldete Version:** `2.12.8-2`

3. [Tor](https://www.torproject.org/download/) von der Tor-Website:

    - **Von Android gemeldete App-Version:** `102.2.1-Release (12.5.6)`
    - **Von der Quelle gemeldete Version:** keine (diese HTML-Quelle liefert keine Versionszeichenfolge, daher wird stattdessen ein URL-Hash als „Pseudoversion“ verwendet)

4. [Quotable](https://github.com/Lijukay/Qwotable/releases/tag/v10) von GitHub:

    - **Von Android gemeldete App-Version:** `1`
    - **Von der Quelle gemeldete Version:** `v10`

Obtainium speichert eine Liste von „Standardformaten“, die es für diesen Vergleich verwendet (wie `x.y.z` oder `x.y`). Wenn beide zu vergleichenden Versionen demselben Format entsprechen, wird der Vergleich durchgeführt. Wenn nicht, wird die Versionserkennung für diese Anwendung deaktiviert. In einigen Fällen entfernt Obtainium zusätzliche Teile aus dem Quelltext, wenn dies zu einer Standardversion führen würde (wie z.B. `v` und `-beta` aus Obtainiums `v0.14.21-beta`), dann kann es den Vergleich durchführen. Wir versuchen nie, Teile der „echten“, vom Betriebssystem bereitgestellten Version zu entfernen.

[Dieser Code](https://github.com/ImranR98/Obtainium/blob/main/lib/providers/apps_provider.dart#L96) definiert, wie die verschiedenen „Standardformate“ generiert werden.

Es ist immer möglich, diesen Code zu erweitern, um Unterstützung für weitere Formate hinzuzufügen, aber dies erfordert sorgfältige Überlegungen. Wenn Android zum Beispiel meldet, dass die installierte Version einer App `1.2` ist, die Quelle aber sagt, dass die neueste verfügbare Version dieser App `1.2-4` ist, sollten wir dann das `-4` entfernen und sagen, dass die beiden gleich sind (was bedeutet, dass kein Update verfügbar ist)? Dies mag in manchen Kontexten in Ordnung sein (wo `-4` nicht auf eine Änderung der App selbst hinweist), in anderen Kontexten jedoch nicht. Daher wäre es keine gute Idee, diesen speziellen Fall zu unterstützen.

Wenn die Versionserkennung deaktiviert ist, sollte dies normalerweise keine signifikanten Auswirkungen auf den täglichen Gebrauch haben. Wenn die Versionserkennung für eine App deaktiviert ist, kann es gelegentlich zu Inkonsistenzen zwischen der tatsächlichen Version der auf Ihrem System installierten App und der in der Obtainium-Benutzeroberfläche angezeigten Version kommen. Dies sollte nur in zwei Fällen geschehen:

1. Wenn sich die Version einer App aufgrund von Aktionen außerhalb von Obtainium ändert (z. B. wenn sie von Google Play aktualisiert wird)
2. Wenn ein Versuch von Obtainium, die App im [Hintergrund](#hintergrundaktualisierungen) zu aktualisieren, fehlschlägt

In solchen Fällen kann Obtainium nicht erkennen, dass sich die tatsächliche Betriebssystemversion der App geändert hat, und würde daher seine internen Aufzeichnungen nicht entsprechend aktualisieren - Sie müssten die Inkonsistenz manuell korrigieren.

Zwei zusätzliche pro-App-Optionen können die Art und Weise beeinflussen, wie Versionen behandelt werden:

- **Versionscode als Betriebssystemversion verwenden**: Wenn aktiviert, verwendet Obtainium den Android-Versionscode (eine Ganzzahl) anstelle des Versionsnamens beim Vergleich mit der Quelle. Dies kann Unstimmigkeiten beheben, wenn die Quelle Versionscodes liefert, das Betriebssystem aber Versionsnamen meldet, oder umgekehrt.
- **Veröffentlichungsdatum als Version**: Wenn diese Option für viele Quellen verfügbar ist, wird das Veröffentlichungsdatum als Pseudoversion anstelle der Versionszeichenfolge der Quelle verwendet. Dies ist nützlich, wenn die Quelle keine aussagekräftigen Versionsinformationen liefert, das Veröffentlichungsdatum aber zuverlässig ist.

Siehe auch: [Obtainium Issue #946 Kommentar](https://github.com/ImranR98/Obtainium/issues/946#issuecomment-1741745587)

## Hintergrundaktualisierungen

Obtainium sucht regelmäßig im Hintergrund nach App-Updates. Die Häufigkeit dieser Aktualisierungsaufgaben können Sie auf der Einstellungsseite festlegen.

### Update-Prüfungsmechanismus

Die Hintergrundaktualisierung kann über einen von zwei Mechanismen erfolgen:

- **WorkManager** (Standard): Verwendet den standardmäßigen Hintergrundaufgabenplaner von Android. Dies ist batterieeffizient, kann aber von einigen OEMs durch ihre Akkuoptimierungsrichtlinien verzögert werden.
- **Vordergrunddienst** (optional): Läuft als dauerhafter Vordergrunddienst mit einer Benachrichtigung. Dies kann auf Geräten, die Hintergrundarbeit aggressiv einschränken, zuverlässiger sein, allerdings mit einer dauerhaften Benachrichtigung. Erfordert Android 11+.

### Update-Einschränkungen

Sie können festlegen, wann Hintergrundüberprüfungen stattfinden:

- **Nur WLAN**: Nur nach Updates suchen, wenn eine WLAN-Verbindung besteht.
- **Nur beim Laden**: Nur nach Updates suchen, wenn das Gerät lädt.

Sie können auch wählen, ob jedes Mal beim Öffnen der App ("Beim Start überprüfen") und/oder beim Anzeigen der Detailseite einer App nach Updates gesucht werden soll.

### Stille Installation

Nach Abschluss einer Hintergrundaktualisierungsprüfung werden alle verfügbaren Updates in 2 Kategorien eingeteilt:

1. Updates, die im Hintergrund installiert werden können
2. Updates, die nicht im Hintergrund installiert werden können

Damit ein Update automatisch im Hintergrund installiert werden kann (auch als stilles Update bezeichnet), müssen bestimmte Kriterien erfüllt sein:

- Das Betriebssystem muss Android 12 oder höher sein
- Die zu installierende App muss auf eine [aktuelle Android-API-Ebene](https://developer.android.com/reference/android/content/pm/PackageInstaller.SessionParams#setRequireUserAction(int)) abzielen
- Die aktuell installierte Version der App muss von Obtainium installiert worden sein
- Sie müssen Hintergrundaktualisierungen in Obtainium aktiviert haben (sowohl universell als auch für diese App - dies ist die Standardeinstellung)
- Wenn mehrere APKs für das Update verfügbar sind, müssen die zusätzlichen Optionen für diese App so konfiguriert sein, dass Obtainium diese auf eine APK reduzieren kann

Stille Installationen können auch mit **Shizuku** durchgeführt werden, das erhöhte Berechtigungen gewährt. Dies kann die Zuverlässigkeit auf einigen Geräten verbessern. Wenn Shizuku aktiviert ist, gibt es eine zusätzliche Option "So tun, als ob Sie Google Play wären", die dem System mitteilt, dass Sie der Google Play Store sind, was bei einigen Apps die Kompatibilität verbessern kann.

Jedes verfügbare Update wird nach Möglichkeit heruntergeladen und installiert, und der Benutzer wird entweder über die Verfügbarkeit des Updates oder darüber benachrichtigt, dass es im Hintergrund installiert wurde.

### Parallele Downloads

Standardmäßig lädt Obtainium Updates nacheinander herunter und installiert sie. Wenn Sie parallele Downloads aktivieren, werden mehrere Updates gleichzeitig verarbeitet, was Bulk-Updates beschleunigen kann, aber mehr Netzwerkbandbreite und Systemressourcen verbraucht.

Beachten Sie, dass Hintergrundaktualisierungen aufgrund technischer Einschränkungen nur auf asynchroner Basis nach bestem Wissen und Gewissen installiert werden können. Wenn also eine Hintergrundaktualisierung nicht installiert werden kann, werden Sie nicht über den Fehler benachrichtigt.
