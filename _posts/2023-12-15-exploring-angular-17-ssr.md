# Exploring Angular 17 Server-side Rendering (SSR)

Language: English

---

Angular 17 was released in November 2023, marking a true renaissance for the framework. It introduces numerous new features, with key highlights being:
- Rebranding
- Built-in Control Flow
- Deferred Loading
- Signals
- Improved Server-side rendering

This article shares my firsthand experience and tests with Server-side Rendering (SSR). Regrettably, as of today, the Angular documentation lacks comprehensive information on how SSR functions.

## Understanding SSR

[Angular Docs] Server-side rendering (SSR) is a process that involves rendering pages on the server, resulting in initial HTML content which contains initial page state. Once the HTML content is delivered to a browser, Angular initializes the application and utilizes the data contained within the HTML.

## Why Opt for SSR?

[Angular Docs] 
The main advantages of SSR as compared to client-side rendering (CSR) are:
- Improved performance: SSR can improve the performance of web applications by delivering fully rendered HTML to the client, which can be parsed and displayed even before the application JavaScript is downloaded. This can be especially beneficial for users on low-bandwidth connections or mobile devices.
- Improved Core Web Vitals: SSR results in performance improvements that can be measured using Core Web Vitals (CWV) statistics, such as reduced First Contentful Paint (FCP) and Largest Contentful Paint (LCP), as well as Cumulative Layout Shift (CLS).
- Better SEO: SSR can improve the search engine optimization (SEO) of web applications by making it easier for search engines to crawl and index the content of the application.

## Setting Up a New SSR Application

To create a new SSR application, use the following command:

`ng new --ssr`

## CSR vs SSR

In client-side rendering (CSR) the browser generates HTML using JavaScript.
In server-side rendering (SSR) the server generates the HTML/CSS for the first request and sends back to the client after that browser download all scripts and Angular starts hydration (non-destructive) process to add interactivity.
After the first request, the application behaves like a normal CSR app.
If the first page includes an HTTP call, the server initiates the call and returns the rendered data to the client, storing it in a transfer state (ng-state).

## Prerendering (SSG)

Angular offers the possibility to pre-render some page at build time.

## Build

I encountered challenges with the build process, as it is not adequately described in the Angular documentation. When compiling your SSR app, the builder creates two folders, each with distinct purposes. Here are my considerations:
- **browser**: contains the classic CSR app and it used by server side to provide static files
- **server**: contains all scripts of which needs to pre-render the first pages called by client. It is similar to technologies like ASP.NET WebForms, where HTML/CSS is rendered by the server.

## Demo

As you can see, the initial response from the first call consists of pure HTML/CSS, and the first component is already rendered by the server. The browser then downloads all scripts from the 'browser' folder to initiate hydration.

![Angular SSR](/images/angular_ssr_first_call.png "Angular SSR")

If I initiate the second component as the initial request, the server will render the second component, along with the necessary scripts for hydration.

![Angular SSR](/images/angular_ssr_first_call_2.png "Angular SSR")

The scripts in the server folder are utilized by the server to render the initially requested page.

As confirmation, I've included the following code in the second component:

```
export class SecondComponent {
  constructor(@Inject(PLATFORM_ID) platformId: Object) {
    console.log('isPlatformServer', isPlatformServer(platformId));
  }
}
```

If the second component is not the first call, I observe isPlatformServer false in the browser console only.

However, if the second component is the first call, I observe isPlatformServer true in the server console and isPlatformServer false in the browser console. This confirms that the page is pre-rendered by the server.

## Download File Example

[GitHub](https://github.com/flecce/angular-ssr-test)