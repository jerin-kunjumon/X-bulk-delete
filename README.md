# X-bulk-delete
Script to delete before a particular date (Credits go to ChatGPT, I didn't write any of this code)

```js
const deleteTweetsBeforeDate = async () => {
  const processed = new Set();
  const MAX_DELETES_PER_RUN = 40;

  // 🔴 CHANGE THIS DATE
  const CUTOFF_DATE = new Date('2026-01-01');

  let deleteCount = 0;

  const selectors = {
    tweet: '[data-testid="tweet"]',
    caret: '[data-testid="caret"]',
    menuItem: '[role="menuitem"]',
    deleteConfirm: '[data-testid="confirmationSheetConfirm"]',
    unretweet: '[data-testid="unretweet"]',
    unretweetConfirm: '[data-testid="unretweetConfirm"]',
    time: 'time'
  };

  const delay = ms => {
    const jitter = ms * 0.4;
    const actual = ms + (Math.random() * jitter * 2 - jitter);
    return new Promise(r => setTimeout(r, actual));
  };

  const scrollToEnd = () =>
    window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' });

  const isBeforeCutoff = tweet => {
    const timeEl = tweet.querySelector(selectors.time);
    if (!timeEl?.dateTime) return false;
    return new Date(timeEl.dateTime) < CUTOFF_DATE;
  };

  const getEligibleButtons = () =>
    Array.from(document.querySelectorAll(selectors.tweet))
      .filter(isBeforeCutoff)
      .map(t => t.querySelector(selectors.caret))
      .filter(b => b && !processed.has(b));

  const attemptDelete = async button => {
    try {
      processed.add(button);
      button.click();
      await delay(600);

      const menuItems = Array.from(document.querySelectorAll(selectors.menuItem));
      const deleteOption = menuItems.find(i =>
        i.textContent?.trim().toLowerCase() === 'delete'
      );

      if (deleteOption) {
        deleteOption.click();
        await delay(600);
        const confirm = document.querySelector(selectors.deleteConfirm);
        if (confirm) {
          confirm.click();
          deleteCount++;
          console.log(`🗑️ Deleted ${deleteCount}/${MAX_DELETES_PER_RUN}`);
          await delay(4000);
          return;
        }
      }

      const tweet = button.closest(selectors.tweet);
      const unretweet = tweet?.querySelector(selectors.unretweet);
      if (unretweet) {
        unretweet.click();
        await delay(600);
        const confirm = document.querySelector(selectors.unretweetConfirm);
        if (confirm) {
          confirm.click();
          deleteCount++;
          console.log(`↩️ Unretweeted ${deleteCount}/${MAX_DELETES_PER_RUN}`);
          await delay(4000);
        }
      }
    } catch (e) {
      console.error(e);
    }
  };

  while (deleteCount < MAX_DELETES_PER_RUN) {
    const buttons = getEligibleButtons();

    if (!buttons.length) {
      scrollToEnd();
      await delay(9000);
      if (!getEligibleButtons().length) break;
      continue;
    }

    for (const btn of buttons) {
      if (deleteCount >= MAX_DELETES_PER_RUN) break;
      await attemptDelete(btn);
      await delay(2000);
    }

    await delay(9000);
    scrollToEnd();
  }

  console.log(
    `✅ Done. ${deleteCount} tweets deleted before ${CUTOFF_DATE.toDateString()}.\n` +
    `⏸️ Stop now and run again tomorrow if needed.`
  );
};

deleteTweetsBeforeDate();



