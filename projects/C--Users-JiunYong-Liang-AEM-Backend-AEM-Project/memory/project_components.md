---
name: AEM Custom Components & Java Backend
description: Inventory of all 13 custom AEM components and core Java backend packages, models, servlets, and services
type: project
originSessionId: 62767ced-1e42-4928-a161-34e6b010f8e3
---
## Custom Components (ui.apps/…/apps/dell/components/custom-components/)

| Component | Purpose |
|-----------|---------|
| course-details-updated | Updated course detail page — SSR via CourseDetailsModel, pricing, schedule table, tabs (Overview/Outline), modals |
| coveosearch | Coveo-powered search — faceted filters (Category, Modality), autocomplete, price range, analytics, DOMPurify |
| course-details | Legacy course detail component |
| available-exams | Certification exam availability display |
| classschedule | Classroom schedule with date/location filtering |
| advancesearchbar | Advanced search UI with multiple filters |
| storesearch | Store/catalog search page |
| mylearningbanner | User profile banner on My Learning page |
| mylearningnav | Navigation for My Learning section |
| mypreferences | User preference management |
| dynamicherolist | Dynamic hero section with course/package listings |
| esupport | Support/help section |
| odlcmodal | Modal for on-demand learning course info |

All components use: HTL templates, Sling Model backing, DDS (Dell Design System) CSS classes, jQuery, DOMPurify, edit dialogs.

## Java Backend Package Map (core/src/main/java/com/dell/core/)

- **bean/** — DTOs: Category, CourseInfoResponse, CoursePayload, FAQData, Offering, Question, Result, SearchResult
- **config/** — DellEmcConfigurations (header/footer paths, PRM URL, dispatcher URL)
- **constants/** — GlobalConstants (200+ constants: URL fragments, widget types, course types, content paths, locales)
- **connection/api/** — REST infrastructure: RESTClientImpl (pooling, SSL, GUID logging), RESTClientConfig, AggregatedServiceImpl
- **connection/api/impl/services/** — CategoryService, GetCourseService, GetOfferingsService, GetPackageService, GetCoursesFromPackage, GenericRequestService, GenericAngularService
- **connection/api/pojo/** — Request/response POJOs: CoursePackageRequest, OfferingCoursePackageRequest, CourseResponse, AggregatedResponse, Rows
- **models/** — Sling Models: CourseDetailsModel (primary SSR), AvailableExamsSsrModel, Multifield, MultifieldJson, NestedMultifield, OfferingView, HelloWorldModel
- **services/** — Business logic: OAuthTokenService, ParseSAMLService, PRMCredentialsService, VoucherCredentialsService, EndPointsServiceImpl, PagePropertySearchService, InternalSearchService, CountryCurrencyService, FAQDataService, GenericRESTCallService, TechDirectService, EmailService, CourseApiClient, EduServiceApiClient
- **servlets/** — HTTP endpoints: SearchServlet, InternalSearchServlet, GetCoursesServlet, GetOfferingCourse, GetCoursesFromPackageServlet, SAMLCallbackServlet, SAMLRedirectServlet, SAMLLogoutServlet, DISamlResponseProcessorServlet, SSOServlet, SabaRestAPIOauth, EMCUsersServlet, RetrieveHeaderVariablesServlet, VerticalCardServlet, VoucherRequestServlet, VoucherGenericServlet, GetVideoPlayList, CashApp/CashApplication/CashApplicationCloud
- **filters/** — LoggingFilter
- **listeners/** — SimpleResourceListener
- **schedulers/** — SimpleScheduledTask, SMPScheduler
- **use/** — HTL use classes: UserIdentity, EdSvcUserInfo, VoucherUserInfo, GetLocale, MultiField, UpdateURLLink, ImageUse
- **utils/** — CourseUtil, CourseDescriptionFormatter, FaqCsvReader, GenericUtil, ParameterUtil
