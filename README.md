# React Tiny Oembed [![version](https://img.shields.io/npm/v/react-tiny-oembed.svg)](https://www.npmjs.com/package/react-tiny-oembed) [![minzipped bundle](https://img.shields.io/bundlephobia/minzip/react-tiny-oembed?label=minzipped%20bundle)](https://www.npmjs.com/package/react-tiny-oembed) 
React component for embedding content from sites going [oEembed](https://oembed.com/) way and only\* oembed way. Just give it a url and it will do the rest, no more paying for widgets!

The motivation behind this component is admiration of oembed, an opensource standard with unified way of embedding content from all supported sites, instead of having different method for every site, no exceptions\*,

> _However sites not supporting oembed for now can also be embedded using oembed-wrapper proxies and interceptors, see `Plugins` below_

## Installation

```bash
npm install react-tiny-oembed
```

_requires React 16.8 or higher_

## Basic usage

```jsx
import Embed from 'react-tiny-oembed'

function App() {
    ...

        <Embed
            url="https://youtu.be/nlD9JYP8u5E"
            proxy="http://cors-anywhere.herokuapp.com/"
        />
}
```

> A note on proxy: most of the the sites do not have cors enabled, so cors proxy is necesary in most cases.
> Above used proxy is just for demonstration and is slow and highly rate limited, so provide your own proxy, you can use host [Cors anywhere](https://github.com/Rob--W/cors-anywhere) on your own node server and use that

By default only _YouTube_, _Reddit_, _Flickr_, _Vimeo_, _SoundCloud_, _Twitter_, _GIPHY_ are enabled, to add more or reduce even default ones, see `providers` prop below

## Props

You can pass multiple props to Embed component, typings can be imported too as named exports.

-   `options`: _EmbedRequestOptions_

    Oject conatining oembed options, these fields are used as query params to oembed provider, these include general options like `maxwidth` and `maxheight` and also some site specific options. below are some default ones used

    | value       | type   | default  | description                                                                                           |
    | ----------- | ------ | -------- | ----------------------------------------------------------------------------------------------------- |
    | `maxwidth`  | number | 700      | maximum value of iframe or image, not container which can bechanged with `style` prop                 |
    | `maxheight` | number | 400      | similar to maxwidth                                                                                   |
    | `align`     | string | 'center' | for [twitter](https://developer.twitter.com/en/docs/twitter-for-websites/timelines/guides/oembed-api) |

-   `style`: _CSSProperties_

    Styles applied to outer container, conatiner also has `__embed` class so you can use that too, by default it takes has `100%` width and `700px` max width

-   `FallbackElement` and `LoadingFallbackElement`: _ReactElement_

    By default the given url is shown as anchor tag (external) for states like loading or error, However you can pass your own ones like

    ```jsx
    <Embed
        options={{ theme: 'dark' }}
        url="https://twitter.com/iamdevloper/status/1324864523363356673"
        proxy="http://cors-anywhere.herokuapp.com/"
        LoadingFallbackElement="Yeah loading..., use your own proxy"
    />
    ```

-   `ImgComponent`: _ComponentType<{ responce?: PhotoEmbedResponce }>_

    While most sites would have their good looking widgets, some sites like _Giphy_ would just give you images. Images are displayed plain, without any styling, you might want to have your own custom component for images. That component will receive `reponce` prop as oembed responce object, you can access `src` via `responce.url`

    ```jsx
        function CustomImg({ responce }) {
            return <div className="img-widget">
                <h1>Image from {responce.provider_name}</h1>
                <img src={responce.url} alt={responce.author_name} />
            </div>
        }

        ...
            <Embed
                ...
                ImgComponent={CustomImg}
            />
    ```

    _similar is for `LinkComponent` but i did not see any site returning just link_

-   `providers` ⭐

    Default providers are just a handful, you have
    hundreds to choose from. This prop can be used to enable (or reduce) support for individual sites. It expects an array of [`Provider`](https://oembed.com/providers.json) objects which defines matching pattern for links, embedding url or interceptors to add to.

    Say you want to extend suppport to more sites, go to [https://oembed.com/providers.json](https://oembed.com/providers.json), choose a provider object, say the first one, pass it.

    ```jsx
        import Embed, { defaultProviders } from 'react-oembed'

        const TwoThreeHQ = {
            "provider_name": "23HQ",
            "provider_url": "http:\/\/www.23hq.com",
            "endpoints": [
                {
                    "schemes": [
                        "http:\/\/www.23hq.com\/*\/photo\/*"
                    ],
                    "url": "http:\/\/www.23hq.com\/23\/oembed"
                }
            ]
        }

        ...

            <Embed
                url= ...

                providers={[...defaultProviders, TwoThreeHQ]}
            />
    ```

    > Note: passing `providers` list overrides default one, so you need to pass `defaultProviders` to have them too.

    Support for all the sites can be extended in this way, just passing list of provider objects. Also remember sites like _Instagram_ and _Facebook_ require developer keys too, so pass them in `options` prop above (please test them, i did not wanted to create developer account there)

    If you want to filter even default ones, you can

    ```js
    const providers = defaultProviders.filter(
        p => p.provider_name === 'Vimeo' || p.provider_name === 'SoundCloud'
    )
    ```

    For sites not supporitng oembed but see `Plugins` section below.

## Plugins

-   [github-gist](https://github.com/muzam1l/oembed-github-gist) - Github gist sample plugin for react-tiny-oembed without a proxy server.
-   ...others

Now we break our top rule, no exceptions, or atleast twist it a bit. While i do believe every site should support oembed instead of having custom method, but there are some who don't and some important ones like _Github_.

But this component still understands _only_ oembed, so the recomended way for those sites is creating a oembed-proxy server, and referencing that in custom provider. Provider objects are self explanatory, important parts of it are `endpoints.schemes` which defines patterns of urls to match and uses _globs_ matched by [minimatch](https://github.com/isaacs/minimatch) and `endpoints.url` which defines oembed comatible url, see [https://oembed.com](https://oembed.com/)

Example of custom provider would look like this

```json
 {
        "provider_name": "My Provider",
        "provider_url": "https://myurl.com/",
        "endpoints": [{
            "schemes": [
                "https://github.com/**"
            ],
            "url": "https://proxyurl.com/{raw_url}",
            "discovery": true
        }]
},
```

> Note: `{raw_url}` and `{url}` are placeholders that can be in `endpoints.url` to specify where the url will be substituted, `{url}` will be replaced by url-encoded url and must be decoded on server accordingly, if both are omitted, url will simply be appended.

### Also

You can also use `requestInterceptor` and `responceInterceptor` fields in Provider object, which should be [axios](https://github.com/axios/axios) interceptor functions whose goal is to convert request into oembed compatible request and responce into oembed comaptible responce, [github-gist](https://github.com/muzam1l/oembed-github-gist) is one such example. But our rule is mostly intact, this component still only understands oembed.

## Contributing

You can help me write tests 😊 or just contribute, i dont have any guidelines.
