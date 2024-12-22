Twitter-Unfollow

 This script automatically unfollows twitter users 

> Go to https://twitter.com/YOUR_USER_NAME/following

> Open the Developer Console. (COMMAND+ALT+I on Mac OR COMMAND+SHIFT+I on Windows)

> Paste this into the Developer Console and run it


(() => {
  const $followButtons = '[data-testid$="-unfollow"]';
  const $confirmButton = '[data-testid="confirmationSheetConfirm"]';

  const retry = {
    count: 0,
    limit: 5,  // Increased retry limit for better coverage
  };

  const scrollToTheBottom = () => window.scrollTo(0, document.body.scrollHeight);
  const retryLimitReached = () => retry.count === retry.limit;
  const addNewRetry = () => retry.count++;

  const sleep = ({ seconds }) =>
    new Promise((proceed) => {
      console.log(`WAITING FOR ${seconds} SECONDS...`);
      setTimeout(proceed, seconds * 1000);
    });

  const unfollowAll = async (followButtons) => {
    console.log(`UNFOLLOWING ${followButtons.length} USERS...`);
    await Promise.all(
      followButtons.map(async (followButton) => {
        if (followButton) {
          followButton.click();
          await sleep({ seconds: 1 });
          const confirmButton = document.querySelector($confirmButton);
          if (confirmButton) confirmButton.click();
        }
      })
    );
  };

  const nextBatch = async () => {
    scrollToTheBottom();
    await sleep({ seconds: 2 });

  const followButtons = Array.from(document.querySelectorAll($followButtons));
    const followButtonsWereFound = followButtons.length > 0;
   
   if (followButtonsWereFound) {
      await unfollowAll(followButtons);
      await sleep({ seconds: 2 });
      return nextBatch();
    } else {
      addNewRetry();
    }

   if (retryLimitReached()) {
      console.log(`NO MORE ACCOUNTS FOUND OR REACHED RETRY LIMIT.`);
      console.log(`RELOAD PAGE AND RE-RUN SCRIPT IF ANY WERE MISSED.`);
    } else {
      await sleep({ seconds: 2 });
      return nextBatch();
    }
  };

  nextBatch();
})();
