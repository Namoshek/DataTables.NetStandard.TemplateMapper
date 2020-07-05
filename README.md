# DataTables.NetStandard.TemplateMapper

This package provides an optional template mapping utility which allows to render HTML columns
when using [DataTables.NetStandard](https://github.com/Namoshek/DataTables.NetStandard).

#### Table of Contents

- [Installation](#installation)
- [Usage](#usage)
- [License](#license)

## Installation

The package can be found on [nuget.org](https://www.nuget.org/packages/DataTables.NetStandard.TemplateMapper/).
You can install the package with:

```pwsh
$> Install-Package DataTables.NetStandard.TemplateMapper
```

## Usage

Before you are able to use the template mapper, you need to register it with your service provider:
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataTablesTemplateMapper();

    // ... register other services ...
}
```

The following example summarizes all options provided by this package:

```csharp
public class DefaultMappingProfile : Profile
{
    public DefaultMappingProfile(IViewRenderService viewRenderService)
    {
        CreateMap<Person, PersonViewModel>()

            // Raw columns containing some HTML (like action buttons) consist of simple strings. This means
            // you can basically add a string column on the view model which does not have to exist on the
            // query model and return some custom HTML for it here in the mapper. In this example we are simply
            // building a link inline. The following two columns do the same but using file-based templates.
            .ForMember(vm => vm.Action, m => m.MapFrom(p => $"<a href=\"#person-{p.Id}\">Link 1</a>"))

            // This uses the package Scriban which parses Liquid templates and renders them with the row data.
            // The Scriban package does not require any dependency injection and offers static methods, which
            // makes it a very easy to use library. The template language Liquid is quite different from Razor
            // though, so it can be a bit of work to get used to it.
            // Probably important: If the row object (person) is passed directly as second argument, its properties
            // will be accessible in the template directly (i.e. <code>p.Id</code> -> <code>{{ id }}</code>).
            // If the row object is wrapped in another object like <code>new { Person = p }</code>, the properties
            // will be accessible with <code>{{ person.id }}</code> for example.
            // Important: Template files have to be copied to the output folder during builds. Make sure this
            //            setting is set correctly in the file properties.
            .ForMember(vm => vm.Action, m => m.MapFrom(p => ViewRenderService.RenderLiquidTemplateFileWithData("DataTables/Person/Action.twig", p)))

            // The same renderer is also available for string based templates instead of file based ones.
            .ForMember(vm => vm.Action, m => m.MapFrom(p => ViewRenderService.RenderLiquidTemplateWithData("<a href=\"#person-{{id}}\">Link 2</a>", p)))

            // This renders the given view as Razor template through the ASP.NET Core MVC Razor engine. Rendering
            // the view this way allows you to use basically all Razor functions available. There is a significant
            // downside to this though: The AutoMapper profile (this class) has to receive the IViewRenderService
            // from the dependency injector somehow, which does not happen by itself and is only possible through
            // a hack in the Startup.ConfigureService() method. Have a look there to learn more about it.
            .ForMember(vm => vm.Action, m => m.MapFrom(p => viewRenderService.RenderRazorToStringAsync("DataTables/Person/ActionColumn", p).Result));
    }
}
```

_Note: When you use dependencies within your mapping profile, you'll have to inject these dependencies
into the profile yourself when initializing your `Mapper`._

## License

The code is licensed under the [MIT license](LICENSE.md).
