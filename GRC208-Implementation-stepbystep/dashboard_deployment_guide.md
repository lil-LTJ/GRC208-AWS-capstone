GRC Dashboard Deployment Guide
The code provided in the assignment (grc-dashboard.jsx and grc-dashboard.css) is written in React, which is the most popular frontend web framework in the world.

Right now, these are just raw code files. To actually turn them into a website you can click on and view in your browser, you have to build a small "React App" container around them to host them.

Bonus note: I checked the internal logic of the code. The author built this dashboard file using mock data. In a real enterprise, these files would connect to an AWS API Gateway to actively pull the live database information we injected earlier. However, because it contains mock data right now, you can spin it up instantly without needing to configure complex AWS internet routing!

Here is exactly how to build this website in 5 minutes:

1. Install Node.js
Just like Python is the engine for backend scripts, Node.js is the engine that runs modern frontend websites like React.

Open your VS Code terminal exactly like before.
Paste this command to download Node.js automatically:
cmd
winget install --id OpenJS.NodeJS.LTS -e --source winget
CRITICAL: Once it finishes, close your terminal with the trash can icon, and open a brand new terminal so it recognizes the newly installed node and npm commands.
2. Create the Website Container (Vite)
We are going to use a tool called Vite, which instantly generates an empty, lightning-fast React website folder.

In your VS Code terminal, make sure you are inside your main project folder (C:\Users\USER\Desktop\Capstone AWS-GRC\GRC208-AWS-Capstone-Project).
Run this command to create a new empty website folder called grc-frontend:
cmd
npx create-vite@latest grc-frontend --template react
(If it asks "Need to install the following packages", type y and hit Enter).
Now, move your terminal inside that brand new folder:
cmd
cd grc-frontend
Tell Node.js to install all the background web dependencies:
cmd
npm install
3. Move the Dashboard Code into the Website
Now we just need to drop the capstone's dashboard files into the heart of the new website container.

In your VS Code file explorer on the left, find the original two files (grc-dashboard.jsx and grc-dashboard.css).
Click and drag both of those files into the newly created grc-frontend/src/ folder.
4. Connect the Wires
Your website needs to know that the dashboard is supposed to be the "Home Page."

Inside that grc-frontend/src/ folder, find the file named App.jsx and open it.
Delete everything currently inside that file.
Paste the following 5 lines of code into App.jsx (this tells the website to load your specific dashboard component):
jsx
import React from 'react'
import GRCDashboard from './grc-dashboard'
function App() {
  return <GRCDashboard />
}
export default App
Save the file! (Ctrl + S)
5. Turn on the Server!
You are officially ready to launch the website.

Go back to your VS Code terminal (make sure you are still inside the grc-frontend folder) and run this command:
cmd
npm run dev
Your terminal will print out a local web address that looks exactly like this: http://localhost:5173/
Hold down Ctrl on your keyboard and click that link! Your web browser will open, and you will instantly see the beautiful, interactive Governance, Risk, and Compliance dashboard that sits on top of all the hard work you did today!
