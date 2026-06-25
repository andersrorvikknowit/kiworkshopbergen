# Oppsummering av funn og prioriterte forbedringstiltak

Basert på møtereferatene `01_sprint-planning.txt` til `08_design-review.txt`.

## Lederoppsummering

Teamet har en tydelig strategisk retning: Payments/Stripe-migreringen er den viktigste tekniske og forretningsmessige prioriteten, og må lande før ferie-/releasefrys i november. Samtidig viser referatene at teamet har flere tilbakevendende operasjonelle problemer som øker risikoen for forsinkelser og feil i checkout: overcommitment, sene design- og content-handoffs, mangelfull release-kontroll, svak observability og uavklart eierskap til test-/tooling.

Checkout-området er både inntektskritisk og teknisk sårbart. Incidenten med Loqate viste at en ekstern avhengighet uten timeout kunne skape omfattende 500-feil i checkout. Teamet har fanget opp de riktige læringspunktene, men tiltakene bør prioriteres og følges opp samlet for å redusere risikoen i payments-migreringen og kommende checkout-forbedringer.

## Viktigste funn

### 1. Payments-migreringen styrer veikartet

Payments/Stripe-migreringen er øverste Q3-prioritet og blokkerer senere arbeid med subscriptions/recurring billing. Stakeholderne er tydelige på at migreringen må være ferdig før november, men ingeniørsiden vurderer at full cutover er for risikabelt uten gradvis innføring.

Funn:

- Shadow mode er valgt som migreringsstrategi.
- Feature flag/kill switch er obligatorisk før produksjonstrafikk.
- Divergens mellom gammel og ny betalingsflyt skal trigge alert ved 0,1%.
- PCI-dokumentasjon må gjennomgås før go-live.
- Load testing og rollback-story er fortsatt åpne eller uferdige.

### 2. Checkout har høy forretningsverdi, men lav feilmargin

Checkout påvirker både direkte inntekt, marketingkampanjer og kommende konverteringsarbeid. Incidenten INC-1847 viste at en enkelt ekstern tjeneste kunne føre til rundt 22 minutter med forhøyede feilrater, med anslått inntektstap på 8-12k USD.

Funn:

- Loqate-kallet manglet timeout og circuit breaker.
- Det fantes ingen canary for checkout-deploy.
- Fredag ettermiddag-deploy bidro til økt operasjonell risiko.
- Observability gjorde det raskt å se at feilraten økte, men tregere å finne årsaken.
- Nytt design for adresseform må ha fallback til manuell inntasting dersom Loqate er treg eller nede.

### 3. Analytics-data er upålitelige

Analytics-events på confirm page fyrer 2-3 ganger i enkelte flyter. Dette påvirker konverteringstallene og skaper støy for growth-teamet.

Funn:

- Feilen har vært kjent over flere møter, men blitt nedprioritert.
- Feilen blokkerer pålitelig måling av checkout conversion improvements.
- Relatert infrastruktur bør vurderes sammen med andre kø-/eventproblemer, som password reset-feilen.

### 4. Teamet overcommitter jevnlig

Flere referater peker på at teamet planlegger for mye arbeid og får carryover mellom sprinter. I retroen stemte teamet frem overcommitment som viktigste forbedringsområde.

Funn:

- Teamet hadde velocity 31, men valgte å committe 24 poeng med buffer i sprintplanleggingen.
- Det er fortsatt carryover og gjentatt diskusjon om å planlegge mot 80% av velocity.
- Uklar "definition of ready" gjør det lettere å dra uferdige eller blokkerte saker inn i sprint.

### 5. Design- og content-handoffs skaper gjentatte blokkere

Adresseform, valideringstilstander, error copy og mobile mocks er avhengige av ferdige handoffs. Priya blir gjentatte ganger blokkert av manglende copy og oppdaterte Figma-leveranser.

Funn:

- Error message-copy mangler over flere møter.
- Design manglet loading, empty og error states for Loqate-autocomplete.
- Accessibility-krav som `aria-describedby` må inn i design og implementering.
- Mobilopplevelsen trenger egen vurdering, inkludert sticky CTA og riktig input mode.

### 6. Test- og tooling-eierskap er uklart

Safari-feil går igjen, BrowserStack-kontoen var utløpt, og ingen hadde tydelig eierskap til fornyelse eller cross-browser testdekning.

Funn:

- Flere bugs er Safari-spesifikke.
- BrowserStack-blokkeren hindret testing på ekte enheter.
- Teamet foreslår en "build cop"-rotasjon for CI, tooling og fornyelser.

### 7. Teamkulturen er et styrkepunkt

Teamet jobber blameless, er ærlige om risiko og lærer aktivt fra hendelser. Det finnes god teknisk dømmekraft i diskusjonene rundt shadow mode, idempotency, rollback, observability og graceful degradation.

Funn:

- Incidenten ble håndtert raskt og ryddig.
- Postmortem førte til konkrete tiltak.
- Retroen identifiserte reelle systemiske problemer, ikke bare enkelthendelser.
- Pairing på webhook-arbeid fungerte godt og bør videreføres.

## Prioriterte forbedringstiltak

Prioriteringen er basert på kombinasjonen av produksjonsrisiko, forretningsverdi, blokkeringseffekt og hvor ofte temaet gjentas i referatene.

| Prioritet | Tiltak | Hvorfor | Foreslått eier | Tidshorisont |
| --- | --- | --- | --- | --- |
| P0 | Legg inn timeouts og circuit breakers på alle eksterne kall i checkout | Hindrer ny kaskadefeil som INC-1847 | Tom | Denne sprinten |
| P0 | Innfør kill switch/feature flag for payments shadow mode | Påkrevd før produksjonstrafikk og rollback | Markus / Raj | Før shadow mode i prod |
| P0 | Fiks analytics-events som fyrer 2-3 ganger | Uten dette kan ikke checkout-konvertering måles pålitelig | Tom | Neste sprint, øverst i backlog |
| P0 | Etabler canary deploy for checkout | Reduserer risiko ved endringer i inntektskritisk flyt | Team lead / "me" | Scope nå, implementer neste sprint |
| P1 | Fullfør PCI-dokumentasjon og sikkerhetsreview | Blocker for payments go-live | Markus / Dinesh | Før go-live |
| P1 | Lag load testing-plan for ny payments-flyt | Avdekker kapasitets- og stabilitetsrisiko før cutover | Markus | Før bredere shadow mode |
| P1 | Legg tracing og bedre observability på address validation | Korter ned tid fra symptom til rotårsak | Sofie | Denne/neste sprint |
| P1 | Håndhev "ingen fredag ettermiddag-deploys" | Incidenten viser at sosial regel ikke er nok | Raj | Forslag innen onsdag |
| P1 | Definer "definition of ready" for sprintarbeid | Reduserer overcommitment og blokkert arbeid | Priya / Raj | Til neste planning |
| P1 | Planlegg sprinter mot ca. 80% av faktisk velocity | Reduserer carryover og gir plass til drift/risiko | Raj | Fra neste planning |
| P2 | Etabler build cop-rotasjon | Gir eierskap til CI, tooling, testkontoer og små blokkere | Raj | Start neste sprint |
| P2 | Forny BrowserStack og styrk cross-browser testing | Safari-feil gjentar seg og blokkerer mobil-/checkoutarbeid | Build cop / Raj | Umiddelbart |
| P2 | Ferdigstill design states for adresseform | Trengs for robust UX ved treg/nede Loqate | Priya | Innen onsdag |
| P2 | Avklar error copy med content-teamet | Blokkerer valideringsflyt og implementering | Priya / Karin | Eskaler om ikke klart tirsdag |
| P2 | Utvid on-call-rotasjonen etter payments-migrering | Reduserer belastning på dagens tre personer | Markus / Team | Etter migreringen |

## Anbefalt gjennomføringsrekkefølge

1. Stabiliser checkout-fundamentet: timeouts, circuit breakers, tracing, Loqate fallback og canary.
2. Sikre payments-migreringen: feature flag, ADR, PCI-review, load test, rollback-plan og divergens-alerting.
3. Rydd målegrunnlaget: analytics-feilen må fikses før checkout conversion-arbeidet vurderes.
4. Reduser planleggingsrisiko: innfør definition of ready og planlegg mot 80% velocity.
5. Tett drifts- og tooling-gap: build cop, BrowserStack, cross-browser testing og tydelig eierskap.

## Risikoer hvis tiltakene ikke prioriteres

- Ny checkout-incident kan gi direkte inntektstap og svekket tillit før viktige kampanjer.
- Payments-migreringen kan bli forsinket eller for risikabel å slippe før november.
- Subscriptions/recurring billing kan skyves ytterligere ut, med potensiell påvirkning på enterprise deals.
- Checkout-konvertering kan optimaliseres på feil datagrunnlag.
- Teamet kan fortsette å akkumulere carryover og operasjonell slitasje.

## Kort konklusjon

Teamet har en sunn kultur og tar gode tekniske valg, men trenger sterkere gjennomføring på operasjonelle grunnprinsipper: robusthet rundt eksterne avhengigheter, trygg release-prosess, pålitelig måling, realistisk sprintplanlegging og tydelig eierskap til tooling. De viktigste tiltakene bør behandles som en samlet risikoreduksjon for payments- og checkout-arbeidet, ikke som løse forbedringspunkter.
