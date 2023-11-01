---
layout: default
title: Upgrading a Demo SPA
parent: Model-View-ViewModel
nav_order: 3
---

# Upgrading a Demo SPA

In this section, we will go through the process of upgrading an existing Next.js to make use of the MVVM architecture.  

The demo application can be cloned from the following [GitHub repository](https://github.com/McMasterRS/lmr_mvvm.git). Make sure that you are on the `main` branch.  

The demo SPA renders a basic shopping list. The user can add items to the list by typing the name of the item in the "Item Name" box and clicking the "Add Item" button. Items are randomly assigned an integer price between $1 CAD and $20 CAD. Clicking the "Delete Last Item" button will delete the last item in the shopping list.  

The items are stored in JSON file located under `json/data.json`.  

## CSS Files

Start by creating a `styles` directory under the root directory of the project. Move the CSS files located in the `app` directory to `styles` directory. Make sure to modify the import statement in `page.tsx` as shown below (if your IDE did not already do that for you):  

```ts
import styles from '../styles/page.module.css'
```

## Components
Create a `components` directory in the root directory of the project. The `components` directory usually contains custom or reusable UI elements. Our SPA uses custom buttons that are declared the `pages.tsx` file. We will move the custom button code to the `components` directory.  

Start by creating a `CustomButton` directory inside the components directory. Create a `CustomButton.tsx` file inside the `CustomButton` directory.  

Move the code responsible for creating custom buttons to the `CustomButton.tsx`:  

```ts
import styled from '@emotion/styled'
import MuiButton, {ButtonProps} from '@mui/material/Button'

// Custom Button
interface CustomButtonProps extends ButtonProps {
    mainColor: string
}

export const CustomButton = styled(MuiButton, {shouldForwardProp: (prop) => prop !== "mainColor"})<CustomButtonProps>(props => ({
    backgroundColor: props.mainColor === 'green' ? '#77DD77' : '#FF6961',
    ':hover': {
        backgroundColor: props.mainColor === 'green' ? '#18A558':'#A80900',
    },
    borderRadius: 28
}));
```

Notice that we need to export the `CustomButton` constant now. We can now import the `CustomButton` component in the `pages.tsx` file:  

```ts
import {CustomButton} from "@/components/CustomButton/CustomButton";
```

The `template.tsx` file contains the code responsible for rendering the light/dark mode switch. We will move this code to a separate `ModeSwitch` component.  

Create a `ModeSwitch` subdirectory inside the components directory with a `ModeSwitch.tsx` file inside it.  

Move the code responsible for rendering the dark/light mode switch to `ModeSwitch.tsx`:  

```ts
import {Box, IconButton} from "@mui/material";
import Brightness7Icon from "@mui/icons-material/Brightness7";
import Brightness4Icon from "@mui/icons-material/Brightness4";
import React from "react";
import {useTheme} from '@mui/material/styles'
import {ColorModeContext} from "@/app/template";

export default function ModeSwitch() {
    const theme = useTheme()
    const colorMode = React.useContext(ColorModeContext)

    return (
        <Box
            sx={{
                display: 'flex',
                width: '100%',
                alignItems: 'center',
                justifyContent: 'center',
                bgcolor: 'background.default',
                color: 'text.primary',
                borderRadius: 1,
                p: 3,
            }}
        >
            {theme.palette.mode.charAt(0).toUpperCase() + theme.palette.mode.slice(1)} Mode
            <IconButton sx={{ ml: 1 }} onClick={colorMode.toggleColorMode} color="inherit">
                {theme.palette.mode === 'dark' ? <Brightness7Icon /> : <Brightness4Icon />}
            </IconButton>
        </Box>
    )
}
```

We can now import the `ModeSwitch` component and use it inside the `template.tsx` file. Your `template.tsx` file should look like this:  

```ts
'use client';

import CssBaseline from "@mui/material/CssBaseline";
import useMediaQuery from "@mui/material/useMediaQuery";
import React from "react";
import {createTheme, ThemeProvider} from '@mui/material/styles'
import themeOptions from "@/config/theme";
import ModeSwitch from "@/components/ModeSwitch/ModeSwitch";

export const ColorModeContext = React.createContext({
    toggleColorMode: () => {},
})

export default function Template({children}: {children?: React.ReactNode} ) {
    const prefersDarkMode = useMediaQuery('(prefers-color-scheme: dark)')

    const [themeMode, setThemeMode] = React.useState<'light' | 'dark' | null>(null)


    const theme = React.useMemo(
        () =>
            createTheme({
                ...themeOptions,
                palette: {
                    mode:
                        themeMode == null
                            ? prefersDarkMode
                                ? 'dark'
                                : 'light'
                            : themeMode,
                },
            }),
        [themeMode, prefersDarkMode]
    )

    const colorMode = React.useMemo(
        () => ({
            toggleColorMode: () => {
                setThemeMode(prevMode => (prevMode == null ? (theme.palette.mode === 'dark' ? 'light' : 'dark') : prevMode === 'light' ? 'dark' : 'light'))
            },
        }),
        [theme]
    )

    return (
        <>
            <ColorModeContext.Provider value={colorMode}>
                <ThemeProvider theme={theme}>
                    <ModeSwitch />
                    <CssBaseline />
                    {children}
                </ThemeProvider>
            </ColorModeContext.Provider>

        </>
    )
}
```

## Create the Model

In the root directory of your project, create a new `models` directory with an `ItemModel.ts` file inside of it. The `ItemModel` should contain the code responsible for fetching items, adding a new item and delete the most recent from the list.  

Add the following code to `ItemModel.ts`:  

``` ts
import {useCallback, useState} from 'react'
import {getAllItems, Item, postItem, removeItem} from "@/app/api/items/item";

const ItemModel = () => {
    const [items, setItems] = useState<Item[]>([]);

    const getItems = useCallback(async () => {
        const fetched_items = await getAllItems()
        if (fetched_items) {
            setItems(fetched_items)
        }
    }, [])

    const addItem = useCallback(async (itemText: string) => {
        if (Array.isArray(items)) {
            const response = await postItem(itemText)

            if (response !== null) {
                const fetched_items = await getAllItems()
                if (fetched_items) {
                    setItems(fetched_items)
                }
            }
        }
    }, [items])

    const deleteLastItem = useCallback(async () => {
        if (Array.isArray(items)) {
            const response = await removeItem()

            if (response !== null) {
                const fetched_items = await getAllItems()
                if (fetched_items) {
                    setItems(fetched_items)
                }
            }
        }
    }, [items])

    return {
        items: items,
        getItems,
        addItem,
        deleteLastItem
    }
}

export default ItemModel
```

## Create the ViewModel

In the MVVM design pattern, the Model should not communicate directly with the View. Instead, we have to make use of a ViewModel to handle modifying the user interface on the View and passing information from the Model to the View.  

Create a `viewmodels` directory in the root directory of your project with an `ItemViewModel.tsx` file inside of it.  

We will move the `useEffect` hook responsible for fetching the items, as well as the `handleChange`, `handleAddItem` and `handleDeleteItem` functions to the ViewModel.  

Add the following lines of code to `ItemViewModel.tsx`:  

```ts
import ItemModel from "@/models/ItemModel";
import {useCallback, useEffect, useState} from "react";

const ItemViewModel = () => {
    const [itemName, setItemName] = useState<string>('');
    const { items, getItems, addItem, deleteLastItem } = ItemModel()

    const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
        if (event.target.value) {
            setItemName(event.target.value);
        }
    }

    const handleAddItem = useCallback(async () => {
        if (itemName) {
            await addItem(itemName)
            setItemName('');
        }
    }, [addItem, itemName])

    const handleDeleteItem = useCallback(async () => {
        await deleteLastItem()

    }, [deleteLastItem])

    useEffect(() => {
        getItems()
    }, [getItems])


    return {
        itemName,
        items,
        handleChange,
        handleAddItem,
        handleDeleteItem,
    }
}

export default ItemViewModel
```

## Update the View

Now that we have proper Model and ViewModel files, we will modify the View by updating the `app/page.tsx` file as shown below:  

```ts
'use client';

import styles from '../styles/page.module.css'
import {Box, Grid, List, ListItem, Stack, TextField, Typography} from "@mui/material";
import {CustomButton} from "@/components/CustomButton/CustomButton";
import ItemViewModel from "@/viewmodels/ItemViewModel";
import {Item} from "@/app/api/items/item";


export default function Home() {
    const {
        itemName,
        items,
        handleChange,
        handleAddItem,
        handleDeleteItem,
    } = ItemViewModel();

    const imgStyle = {
        paddingTop: '10px',
        paddingBottom: '10px',
        paddingRight: '30px',
    }

    return (
        <main className={styles.main}>
            <>
                <Stack direction={"column"} spacing={2}>
                    <div style={{
                        display: 'flex',
                        alignItems: 'center',
                        justifyContent: 'center',
                    }}>
                        <Box
                            sx={{
                                height: '50%',
                                width: '50%',
                            }}
                            component="img"
                            alt="Shopping Cart Logo"
                            src="/assets/shopping-cart.png"
                            style={imgStyle}
                        />
                    </div>
                    <TextField id="item-field" label="Item Name" variant="outlined" value={itemName} onChange={handleChange} />
                    <Stack direction={"row"} spacing={2}>
                        <CustomButton mainColor={"green"} variant={"contained"} onClick={handleAddItem}>
                            Add Item
                        </CustomButton>
                        <CustomButton mainColor={"red"} variant={"contained"} onClick={handleDeleteItem}>
                            Delete Last Item
                        </CustomButton>
                    </Stack>
                    <div>
                        <Typography variant={"h1"}>My Shopping List</Typography>
                        <List sx={{ listStyleType: 'disc', pl: 4 }}>
                            {items.map((item: Item) => (
                                <ListItem sx={{ display: 'list-item' }} key={item.id}>
                                    <Grid container spacing={2}>
                                        <Grid item xs={5}>
                                            {item.name}
                                        </Grid>
                                        <Grid item xs={4}>
                                            {"$" + item.price + " CAD"}
                                        </Grid>
                                    </Grid>
                                </ListItem>
                            ))}
                        </List>
                    </div>

                </Stack>
            </>
        </main>
    )
}
```

We removed the code responsible for creating the `CustomButton` component. The `CustomButton` component is now imported from the `CustomButton/CustomButton.tsx` file in the `components` directory.
We also removed the `useEffect` hook responsible for fetching the items, as well as the `handleChange`, `handleAddItem` and `handleDeleteItem` function declarations and replaced them with a call to the `ItemViewModel`.  

Notice that the `ItemModel` is not used directly in the View since the View should not communicate directly with the View.  
