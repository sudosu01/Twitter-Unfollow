Twitter-Unfollow

 This script automatically unfollows twitter users 

> Go to https://twitter.com/YOUR_USER_NAME/following

> Open the Developer Console. (COMMAND+ALT+I on Mac OR COMMAND+SHIFT+I on Windows)

> Paste this into the Developer Console and run it


    (() => {
      // Selectors for the unfollow button and the confirmation button
      const $followButtons = '[data-testid$="-unfollow"]';
      const $confirmButton = '[data-testid="confirmationSheetConfirm"]';
    
      // Retry settings: limits the number of retries to avoid infinite loops
      const retry = {
        count: 0,
        limit: 5,  // Increased retry limit for better coverage
      };
    
      // Function to scroll the page to the bottom to load more users
      const scrollToTheBottom = () => window.scrollTo(0, document.body.scrollHeight);
    
      // Check if retry limit has been reached
      const retryLimitReached = () => retry.count === retry.limit;
    
      // Increment the retry count after each failed attempt
      const addNewRetry = () => retry.count++;
    
      // Helper function to simulate waiting (sleep) for a specified number of seconds
      const sleep = ({ seconds }) =>
        new Promise((proceed) => {
          console.log(`WAITING FOR ${seconds} SECONDS...`);
          setTimeout(proceed, seconds * 1000);
        });
    
      // Function to click the unfollow buttons and confirm unfollowing
      const unfollowAll = async (followButtons) => {
        console.log(`UNFOLLOWING ${followButtons.length} USER(S)...`);
    
        // Loop through all the unfollow buttons and click each one
        await Promise.all(
          followButtons.map(async (followButton) => {
            if (followButton) {
              followButton.click();  // Click the unfollow button
              await sleep({ seconds: 1 });  // Wait for 1 second before confirming
              
              // Look for the confirmation button and click it if available
              const confirmButton = document.querySelector($confirmButton);
              if (confirmButton) {
                confirmButton.click();  // Confirm the unfollow action
              }
            }
          })
        );
      };
    
      // Function to scroll, check for follow buttons, and process them in batches
      const nextBatch = async () => {
        scrollToTheBottom();  // Scroll to the bottom to load more users
        await sleep({ seconds: 2 });  // Wait for new content to load
    
        // Get all available unfollow buttons
        const followButtons = Array.from(document.querySelectorAll($followButtons));
        
        // If there are unfollow buttons, process them
        if (followButtons.length > 0) {
          await unfollowAll(followButtons);  // Unfollow all found users
          await sleep({ seconds: 2 });  // Wait before processing the next batch
          return nextBatch();  // Process the next batch
        } else {
          addNewRetry();  // If no buttons were found, retry
        }
    
        // If the retry limit is reached, log a message and stop
        if (retryLimitReached()) {
          console.log(`NO MORE ACCOUNTS FOUND OR REACHED RETRY LIMIT.`);
          console.log(`RELOAD PAGE AND RE-RUN SCRIPT IF ANY WERE MISSED.`);
        } else {
          // If the retry limit hasn't been reached, continue the process
          await sleep({ seconds: 2 });
          return nextBatch();
        }
      };
    
      // Start the unfollowing process
      nextBatch();
    })();
