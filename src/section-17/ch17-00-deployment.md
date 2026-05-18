# Getting React Apps Live

Building a React app is one thing. Getting it on the internet where people can actually use it is another. For a long time, deployment felt like this mysterious step that I kept putting off. But it does not have to be scary.

Deployment is really just taking your code and putting it on a computer that stays on all the time. That computer is called a server, and it serves your app to anyone who visits your URL.

In this section, I will share what I have learned about deploying React apps. We will cover:

- The build process and what actually happens when you run `npm run build`
- Deploying to Vercel (the easiest option for React and Next.js)
- Deploying to Netlify (another great platform with some nice extras)
- Using Docker for more control over your deployment
- Managing environment variables across different stages
- Setting up CI/CD so your app deploys automatically when you push code

The goal is not to master every deployment method. It is to understand the basics well enough that you can get your app live with confidence. Once you have done it a few times, it becomes second nature.

I remember the first time I deployed an app. I was nervous something would break. It did break, actually. But fixing it taught me more than any tutorial could. So do not worry if your first deployment is not perfect. That is how you learn.

Let us start by understanding what happens when you build a React app.
