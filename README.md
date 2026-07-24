# EMC NEMS Data Services Model Definitions

![GitHub Release](https://img.shields.io/github/v/release/jon-bong/emc-sewplus-nems-models)
[![NuGet Version (Emc.SewPlus.Nems.Models)](https://img.shields.io/nuget/v/Emc.SewPlus.Nems.Models.svg?style=flat-square)](https://www.nuget.org/packages/Emc.SewPlus.Nems.Models/)
![NuGet Downloads](https://img.shields.io/nuget/dt/Emc.SewPlus.Nems.Models)
![GitHub License](https://img.shields.io/github/license/jon-bong/emc-sewplus-nems-models)

Model definitions for data services provided by the National Electricity Market of Singapore (NEMS) system of the Energy Market Company (EMC) of Singapore.

Definitions are based on the [Data Services Specification for NEMS System](https://www.sewplus.emcsg.com/market-system/data-services/data-services-specification) document.

## 🚀 Key Features
- Classes in this package represent EMC Download Reports or Submission classes whose objects contain data that is handled by EMC Data Services in the form of serialised (XML) data.
- Use of enumerations to represent object properties that take on a finite set of `string` values.
- EMC Download Reports classes can be serialised/deserialised using either their individual types or their report category types.

## 📦 Installation
Install the package via the NuGet Package Manager Console, the Nuget Package Manager UI, the .NET CLI or by adding a package reference.

### .NET CLI
```bash
dotnet add package EmcSG.SewPlus.Nems.Models-x.x.x.nupkg
```

### Package Manager
```powershell
Install-Package EmcSG.SewPlus.Nems.Models-x.x.x.nupkg
```

## 🛠️ Quick Start & Usage
All classes representing data elements in the EMC Data Services are marked as `Serializable`, thereby allowing themselves to participate in XML serialisation and deserialisation.

### Deserialising XML Data from an EMC Web Service
Suppose the following real time price data is returned (as a XML string) and saved in a file *RTP.xml*:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<list>
    <RealTimePrice>
        <period>25</period>
        <reportType>REP</reportType>
        <tradingDate>31-Jul-2022</tradingDate>
        <demand>6101.455</demand>
        <tcl>0.000</tcl>
        <USEP>183.08</USEP>
        <lcp>0.00</lcp>
        <regulation>28.15</regulation>
        <primaryReserve>1</primaryReserve>
        <secondaryReserve/>
        <contingencyReserve>20.16</contingencyReserve>
        <eheur>-0.76</eheur>
    </RealTimePrice>
    <RealTimePrice>
        <period>26</period>
        <reportType>REP</reportType>
        <tradingDate>31-Jul-2022</tradingDate>
        <demand>6069.125</demand>
        <tcl>0.000</tcl>
        <USEP>160.15</USEP>
        <lcp>0.00</lcp>
        <regulation>10</regulation>
        <primaryReserve>.14</primaryReserve>
        <secondaryReserve/>
        <contingencyReserve>2</contingencyReserve>
        <eheur>-0.66</eheur>
    </RealTimePrice>
</list>
```

To deserialise the XML data into a `RealTimePrice` object:

```csharp

using Emc.SewPlus.Nems.Models.Reports.Cwr;
using System.Xml;
using System.Xml.Serialization;

public List<RealTimePrice> DeserializeRealTimePriceXml(string xmlFile)
{
    XmlSerializer serializer = new XmlSerializer(typeof(List<RealTimePrice>), new XmlRootAttribute("list"));
    using (FileStream fileStream = new FileStream(xmlFile, FileMode.Open, FileAccess.Read))
    {
        using (XmlReader xmlReader = XmlReader.Create(fileStream))
        {
            var results = serializer.Deserialize(xmlReader) as List<RealTimePrice>;
            return results;
        }
    }
}

```
To deserialise the same data into the same object using the child elements:

```csharp

using Emc.SewPlus.Nems.Models.Reports.Cwr;
using System.Linq;
using System.Xml.Linq;

public List<RealTimePrice> DeserializeRealTimePriceXml(string xmlFile)
{
    XDocument doc = XDocument.Load(xmlFile);
    var results = doc.Element("list").Elements("RealTimePrice")
        .Select(n => new RealTimePrice()
        {
            Period = byte.Parse(n.Element("period").Value),
            ReportType = n.Element("reportType").Value,
            TradingDateString = n.Element("tradingDate").Value,
            Demand = decimal.Parse(n.Element("demand").Value),
            TotalCurtailedLoad = decimal.Parse(n.Element("tcl").Value),
            Usep = decimal.Parse(n.Element("USEP").Value),
            LoadCurtailmentPrice = decimal.Parse(n.Element("lcp").Value),
            RegulationPrice = decimal.Parse(n.Element("regulation").Value),
            PrimaryReservePrice = decimal.Parse(n.Element("primaryReserve").Value),
			
            // property is nullable (defined as nillable in XSD)
            SecondaryReservePrice = (n.Element("secondaryReserve").Value == null)? null : n.Element("secondaryReserve").Value,
						
            ContingencyReservePrice = n.Element("contingencyReserve").Value,
            Eheur = decimal.Parse(n.Element("eheur").Value)
        }).ToList();

    return results;
}

```

#### EMC Download Reports Category Classes
This package also offers a set of classes under the `Emc.SewPlus.Nems.Models.Reports` namespace that simplifies the deserialisation of XML data without having to specify the exact report class. Their names end with _Reports_:

- `Emc.SewPlus.Nems.Models.Reports.Cwr.CorporateWebsiteReports` representing the set of *Corporate Website Reports*.
- `Emc.SewPlus.Nems.Models.Reports.Mcr.MarketClearingRunReports` representing the set of *Market Clearing Run Reports*.
- `Emc.SewPlus.Nems.Models.Reports.Stl.SettlementReports` representing the set of *Settlement Reports*.
- `Emc.SewPlus.Nems.Models.Reports.Tsr.TradeSummaryReports` representing the set of *Trade Summary Reports*.
- `Emc.SewPlus.Nems.Models.Reports.Opr.OtherPublishedReports` representing the set of *Other Published Reports*.

Using the same file *RTP.xml*, its contents can be deserialised using the `CorporateWebsiteReports` class since the corresponding Real Time Price report comes under the set of Corporate Website reports:
```csharp

using Emc.SewPlus.Nems.Models.Reports.Cwr;
using System.Xml;
using System.Xml.Serialization;

public List<RealTimePrice> DeserializeRealTimePriceXml(string xmlFile)
{
    // use the CorporateWebsiteReports class to deserialise the XML data instead of the RealTimePrice class
    XmlSerializer serializer = new XmlSerializer(typeof(CorporateWebsiteReports));
    using (FileStream fileStream = new FileStream(xmlFile, FileMode.Open, FileAccess.Read))
    {
        using (XmlReader xmlReader = XmlReader.Create(fileStream))
        {
            var results = serializer.Deserialize(xmlReader) as CorporateWebsiteReports;
            return results.Items.Cast<Emc.SewPlus.Nems.Models.Reports.Cwr.RealTimePrice>().ToList();
        }
    }
}

```

### Serialising XML Data for an EMC Web Service
To serialise an object of a class e.g. `BidSubmission` into XML to submit to a web service call:

```csharp

using Emc.SewPlus.Nems.Models.Submissions.Bids;
using System.Xml;
using System.Xml.Serialization;

public string SerializeBidSubmissionToXml(BidSubmission bidSubmission)
{
    XmlSerializer serializer = new XmlSerializer(typeof(BidSubmission));
    using (StringWriter writer = new StringWriter())
    {
        serializer.Serialize(writer, bidSubmission);
        return writer.ToString();
    }
}

```

## 🚀 Target Frameworks
* **.NET:** 5.0, 6.0, 7.0, 8.0, 9.0
* **.NET Framework:** 4.0, 4.5, 4.5.2, 4.6.1, 4.6.2, 4.7.2, 4.8, 4.8.1
* **.NET Standard:** 2.0, 2.1, 3.0, 3.1

## 🗺️ Roadmap
- **[] Planned:** Incorporate JSON serialisation and deserialisation.

_Have a feature request? Please open a [feature suggestion](#feedback-and-support)._

## 🤝 Feedback and Support
User feedbacks, bug reports and feature requests are welcome! Since the core codebase is private, please use the following channels to get in touch:
- Bug Reports & Feature Requests: Please open an issue directly on our GitHub Issues Tracker.
- Discussions and Questions: send me an [e-mail](##author-and-contact).

## 👨‍💻 Author and Contact
* **Maintainer:** Jonathan Bong
* **E-mail:** [jonbong1607@hotmail.com](mailto:jonbong1607@hotmail.com)
* [![LinkedIn](https://img.shields.io/badge/linkedin-%230077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/jonathan-bong-5a229840/)
* [![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColor=white)](https://github.com/jon-bong)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
