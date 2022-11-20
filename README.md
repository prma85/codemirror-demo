# Build an editor using CodeMirror with linting and autocomplete

### 1. Before you begin

Build alongside us using Stackblitz `https://stackblitz.com/` and create a React instance

Or use your own environment and make sure you have `node` and `yarn` installed to get started.

### 2. Install the main react-codemirror package

```
yarn add @uiw/react-codemirror

yarn addd @codemirror/lang-javascript
```

### 3. Your basic setup to accept JavaScript

```
import React from 'react';
import CodeMirror from '@uiw/react-codemirror';
import { javascript } from '@codemirror/lang-javascript';

export default function App() {
  const onChange = React.useCallback((value, viewUpdate) => {
    console.log('value:', value);
  }, []);
  return (
    <CodeMirror
      value="console.log('hello world!');"
      height="200px"
      extensions={[javascript()]}
      onChange={onChange}
    />
  );
}
```

### 4. Add multiple editors supporting HTML and CSS

```
import React from 'react';
import './style.css';

import CodeMirror from '@uiw/react-codemirror';
import { javascript } from '@codemirror/lang-javascript';
import { html } from '@codemirror/lang-html';
import { css } from '@codemirror/lang-css';

export default function App() {
  const onChange = React.useCallback((value, viewUpdate) => {
    console.log('value:', value);
  }, []);
  return (
    <div>
      <div>
        <CodeMirror
          value="console.log('hello world!');"
          height="200px"
          extensions={[javascript({})]}
          onChange={onChange}
        />
      </div>
      <div>
        <CodeMirror
          value="<html> </html>"
          height="200px"
          extensions={[html()]}
          onChange={onChange}
        />
      </div>
      <div>
        <CodeMirror
          value="h1 {}"
          height="200px"
          extensions={[css()]}
          onChange={onChange}
        />
      </div>
    </div>
  );
}

```

### 5. Add pre-installed themes and customize others

Install some prebuild themes to play around with 

```
import { sublime } from '@uiw/codemirror-theme-sublime';
import { githubDark } from '@uiw/codemirror-theme-github';
import { okaidia } from '@uiw/codemirror-theme-okaidia';
```

### 6. Add your own custom theme

Create a new file called `customTheme.js`

```
import { createTheme } from '@uiw/codemirror-themes';
import { tags as t } from '@lezer/highlight';

const myTheme = createTheme({
  theme: 'light',
  settings: {
    background: '#ffffff',
    foreground: '#75baff',
    caret: '#5d00ff',
    selection: '#036dd626',
    selectionMatch: '#036dd626',
    lineHighlight: '#8a91991a',
    gutterBackground: '#fff',
    gutterForeground: '#8a919966',
  },
  styles: [
    { tag: t.comment, color: '#787b8099' },
    { tag: t.variableName, color: '#0080ff' },
    { tag: [t.string, t.special(t.brace)], color: '#5c6166' },
    { tag: t.number, color: '#5c6166' },
    { tag: t.bool, color: '#5c6166' },
    { tag: t.null, color: '#5c6166' },
    { tag: t.keyword, color: '#5c6166' },
    { tag: t.operator, color: '#5c6166' },
    { tag: t.className, color: '#5c6166' },
    { tag: t.definition(t.typeName), color: '#5c6166' },
    { tag: t.typeName, color: '#5c6166' },
    { tag: t.angleBracket, color: '#5c6166' },
    { tag: t.tagName, color: '#5c6166' },
    { tag: t.attributeName, color: '#5c6166' },
  ],
});

export default myTheme;
```

### 7. Save to localStorage with undo history

```
import { historyField } from '@codemirror/commands';

// See [toJSON](https://codemirror.net/docs/ref/#state.EditorState.toJSON) documentation for more details
const stateFields = { history: historyField };

export default function App() {
  const serializedState = localStorage.getItem('myEditorState');
  const value = localStorage.getItem('myValue') || '';

  return (
    <CodeMirror
      value={value}
      height={'300px'}
      theme={githubDark}
      extensions={[javascript({})]}
      initialState={
        serializedState
          ? {
              json: JSON.parse(serializedState || ''),
              fields: stateFields,
            }
          : undefined
      }
      onChange={(value, viewUpdate) => {
        localStorage.setItem('myValue', value);

        const state = viewUpdate.state.toJSON(stateFields);
        localStorage.setItem('myEditorState', JSON.stringify(state));
      }}
    />
  );
}
```

### 7. Add your own linter

Create an export a file called `lint.js` and paste the following into it (then update depedencies)

```
TBD
```

Afterwards, update your main App.js code 

```
import React from 'react';
import './style.css';

import CodeMirror from '@uiw/react-codemirror';
import { javascript } from '@codemirror/lang-javascript';
import { githubDark } from '@uiw/codemirror-theme-github';
import { historyField } from '@codemirror/commands';
import { lintGutter } from '@codemirror/lint';

import { jsLinter } from './linter';

// See [toJSON](https://codemirror.net/docs/ref/#state.EditorState.toJSON) documentation for more details
const stateFields = { history: historyField };

export default function App() {
  const serializedState = localStorage.getItem('myEditorState');
  const value = localStorage.getItem('myValue') || '';

  const lintOptions = {
    esversion: 11,
    browser: true,
  };

  return (
    <CodeMirror
      value={value}
      height={'300px'}
      theme={githubDark}
      extensions={[javascript({}), jsLinter(lintOptions), lintGutter()]}
      initialState={
        serializedState
          ? {
              json: JSON.parse(serializedState || ''),
              fields: stateFields,
            }
          : undefined
      }
      onChange={(value, viewUpdate) => {
        localStorage.setItem('myValue', value);

        const state = viewUpdate.state.toJSON(stateFields);
        localStorage.setItem('myEditorState', JSON.stringify(state));
      }}
    />
  );
}
```

### 8. Add your own autocomplete

Create a new file called `completions.js` and put the following code inside of it

```
import { syntaxTree } from '@codemirror/language';

const tagOptions = [
  'constructor',
  'deprecated',
  'link',
  'param',
  'returns',
  'type',
].map((tag) => ({ label: '@' + tag, type: 'keyword' }));

export function completeJSDoc(context) {
  let nodeBefore = syntaxTree(context.state).resolveInner(context.pos, -1);
  if (
    nodeBefore.name != 'BlockComment' ||
    context.state.sliceDoc(nodeBefore.from, nodeBefore.from + 3) != '/**'
  )
    return null;
  let textBefore = context.state.sliceDoc(nodeBefore.from, context.pos);
  let tagBefore = /@\w*$/.exec(textBefore);
  if (!tagBefore && !context.explicit) return null;
  return {
    from: tagBefore ? nodeBefore.from + tagBefore.index : context.pos,
    options: tagOptions,
    validFor: /^(@\w*)?$/,
  };
}
```

Your main code should look like this now

```
import React from 'react';
import './style.css';

import CodeMirror from '@uiw/react-codemirror';
import { javascript, javascriptLanguage } from '@codemirror/lang-javascript';
import { githubDark } from '@uiw/codemirror-theme-github';
import { historyField } from '@codemirror/commands';
import { lintGutter } from '@codemirror/lint';

import { jsLinter } from './linter';
import { completeJSDoc } from './jsDocCompletion';

// See [toJSON](https://codemirror.net/docs/ref/#state.EditorState.toJSON) documentation for more details
const stateFields = { history: historyField };

export default function App() {
  const serializedState = localStorage.getItem('myEditorState');
  const value = localStorage.getItem('myValue') || '';

  const lintOptions = {
    esversion: 11,
    browser: true,
  };

  const jsDocCompletions = javascriptLanguage.data.of({
    autocomplete: completeJSDoc,
  });

  return (
    <CodeMirror
      value={value}
      height={'300px'}
      theme={githubDark}
      extensions={[
        javascript({}),
        jsLinter(lintOptions),
        lintGutter(),
        jsDocCompletions,
      ]}
      initialState={
        serializedState
          ? {
              json: JSON.parse(serializedState || ''),
              fields: stateFields,
            }
          : undefined
      }
      onChange={(value, viewUpdate) => {
        localStorage.setItem('myValue', value);

        const state = viewUpdate.state.toJSON(stateFields);
        localStorage.setItem('myEditorState', JSON.stringify(state));
      }}
    />
  );
}

```


### 9. Let's add a completion for the entire global scope

Createa a file called `completeGlobalScope.js`

```
import { syntaxTree } from '@codemirror/language';
import { CompletionContext } from '@codemirror/autocomplete';

const completePropertyAfter = ['PropertyName', '.', '?.'];
const dontCompleteIn = [
  'TemplateString',
  'LineComment',
  'BlockComment',
  'VariableDefinition',
  'PropertyDefinition',
];

export function completeFromGlobalScope(context: CompletionContext) {
  let nodeBefore = syntaxTree(context.state).resolveInner(context.pos, -1);

  if (
    completePropertyAfter.includes(nodeBefore.name) &&
    nodeBefore.parent?.name == 'MemberExpression'
  ) {
    let object = nodeBefore.parent.getChild('Expression');
    if (object?.name == 'VariableName') {
      let from = /\./.test(nodeBefore.name) ? nodeBefore.to : nodeBefore.from;
      let variableName = context.state.sliceDoc(object.from, object.to);
      if (typeof window[variableName] == 'object')
        return completeProperties(from, window[variableName]);
    }
  } else if (nodeBefore.name == 'VariableName') {
    return completeProperties(nodeBefore.from, window);
  } else if (context.explicit && !dontCompleteIn.includes(nodeBefore.name)) {
    return completeProperties(context.pos, window);
  }
  return null;
}

function completeProperties(from: number, object: Object) {
  let options = [];
  for (let name in object) {
    options.push({
      label: name,
      type: typeof object[name] == 'function' ? 'function' : 'variable',
    });
  }
  return {
    from,
    options,
    validFor: /^[\w$]*$/,
  };
}

````
