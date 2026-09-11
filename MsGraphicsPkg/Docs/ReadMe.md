# MS Graphics Package

## About

This package has shared drivers and libraries that extend the UEFI graphics with a simple windowing system.

## UEFI Driver Model Dependencies

The following graph shows the protocol-mediated dependencies between the graphics stack, the UEFI console drivers,
and FrontPage. `MsGopOverrideDxe` is named `GopOverrideDxe` in this source tree. Solid arrows show protocols consumed
through the UEFI driver model; dashed arrows show linked library use and console I/O exposed through the EFI System
Table.

```mermaid
flowchart TD
	PlatformGop[Platform GOP driver] -->|produces| PhysicalGop{{EFI_GRAPHICS_OUTPUT_PROTOCOL}}

	subgraph GraphicsStack[MS graphics stack]
		MsGopOverride[MsGopOverrideDxe<br/>GopOverrideDxe] -->|publishes original GOP as| OverrideGop{{PcdMsGopOverrideProtocolGuid}}
		OverrideGop -->|controller to start| RenderingEngine[RenderingEngineDxe]
		RenderingEngine -->|produces| RenderedGop{{EFI_GRAPHICS_OUTPUT_PROTOCOL}}
		RenderingEngine -->|produces| Sre{{MS_RENDERING_ENGINE_PROTOCOL}}
		RenderedGop -->|consumes| WindowManager[SimpleWindowManagerDxe]
		Sre -->|consumes| WindowManager
		WindowManager -->|produces| Swm{{MS_SIMPLE_WINDOW_MANAGER_PROTOCOL}}
		SimpleUiToolkit[SimpleUIToolKit<br/>UIToolKitLib]
		RenderedGop -->|consumes| SimpleUiToolkit
		Swm -->|consumes| SimpleUiToolkit
		HiiFont{{EFI_HII_FONT_PROTOCOL}} -->|consumes| SimpleUiToolkit
		Osk{{MS_ONSCREEN_KEYBOARD_PROTOCOL}} -->|edit boxes consume| SimpleUiToolkit
		WindowManager -.->|links| SimpleUiToolkit
	end

	PhysicalGop -->|controller to start| MsGopOverride

	subgraph ConsoleStack[UEFI console stack]
		RenderedGop -->|controller to start| GraphicsConsole[GraphicsConsoleDxe]
		GraphicsConsole -->|produces| TextOut{{EFI_SIMPLE_TEXT_OUTPUT_PROTOCOL}}
		TextOut -->|classifies console handles| ConPlatform[ConPlatformDxe]
		RenderedGop -->|optionally classifies display| ConPlatform
		ConPlatform -->|installs| ConsoleOut{{EFI_CONSOLE_OUT_DEVICE_GUID}}
		TextOut -->|aggregates output| ConSplitter[ConSplitterDxe]
		ConsoleOut -->|selects output handles| ConSplitter
		RenderedGop -->|optionally aggregates GOP| ConSplitter
		ConSplitter -->|produces virtual console| SystemTable{{EFI System Table Console I/O}}
	end

	Swm -->|consumes| FrontPage[FrontPage application]
	RenderedGop -->|sometimes consumes| FrontPage
	FrontPage -.->|links| SimpleUiToolkit
	SystemTable -.->|console I/O| FrontPage

	classDef driver fill:#dbeafe,stroke:#1d4ed8,color:#172554
	classDef library fill:#fef3c7,stroke:#b45309,color:#78350f
	classDef protocol fill:#dcfce7,stroke:#15803d,color:#14532d
	classDef external fill:#f3f4f6,stroke:#4b5563,color:#111827
	class MsGopOverride,RenderingEngine,WindowManager,GraphicsConsole,ConPlatform,ConSplitter,FrontPage driver
	class SimpleUiToolkit library
	class PhysicalGop,OverrideGop,RenderedGop,Sre,Swm,HiiFont,Osk,TextOut,ConsoleOut,SystemTable protocol
	class PlatformGop external
```

## Copyright

Copyright (C) Microsoft Corporation. All rights reserved.
SPDX-License-Identifier: BSD-2-Clause-Patent
