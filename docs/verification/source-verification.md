# Source Verification Audit Trail

検証日: 2026-07-17
実施方法: aidd:source-verifier agent (WebSearch + WebFetch) による外部照合。掲載ルール「検証できない出典・数値は掲載禁止」の監査証跡。

対象: `skills/requirements-notation/references/*.md` および `skills/ambiguity-lint/SKILL.md` の「科学的根拠」全9出典。

| # | 出典 | 判定 | 根拠 URL | 検索クエリ |
|---|---|---|---|---|
| 1 | Mavin, Wilkinson, Harwood, Novak (2009). "Easy Approach to Requirements Syntax (EARS)". RE'09, pp. 317-322. DOI: 10.1109/RE.2009.9 | 確認済み (著者・会議・ページ・DOI とも CrossRef で完全一致) | https://api.crossref.org/works/10.1109/RE.2009.9 / https://research.manchester.ac.uk/en/publications/easy-approach-to-requirements-syntax-ears/ | `Mavin Wilkinson Harwood Novak "Easy Approach to Requirements Syntax" EARS RE'09 DOI 10.1109/RE.2009.9` |
| 2 | Mavin, Wilkinson (2010). "Big Ears". RE'10, pp. 277-282. DOI: 10.1109/RE.2010.39 | 確認済み (CrossRef + IEEE Xplore document 5636542。DOI・ページ番号の食い違いなし) | https://api.crossref.org/works/10.1109/RE.2010.39 / https://ieeexplore.ieee.org/document/5636542/ | `Mavin Wilkinson "Big Ears" RE'10 DOI 10.1109/RE.2010.39` |
| 3 | North (2006). "Introducing BDD". Better Software Magazine | 確認済み。ただし掲載 URL は現行パス https://dannorth.net/blog/introducing-bdd/ が正 → reference を修正済み | https://dannorth.net/blog/introducing-bdd/ / https://dannorth.net/2006/03/19/bdd-article-published-in-better-software-magazine/ | `Dan North "Introducing BDD" Better Software Magazine 2006` |
| 4 | Gherkin Reference (Cucumber 公式) | 確認済み | https://cucumber.io/docs/gherkin/reference/ | `Gherkin Reference cucumber.io docs` |
| 5 | Wake (2003). "INVEST in Good Stories, and SMART Tasks" (2003-08-17) | 確認済み。ただし到達可能な URL は `/articles/` を含まない https://xp123.com/invest-in-good-stories-and-smart-tasks/ → reference を修正済み | https://xp123.com/invest-in-good-stories-and-smart-tasks/ | `Bill Wake "INVEST in Good Stories, and SMART Tasks" 2003 xp123.com` |
| 6 | Cohn (2004). "User Stories Applied". Addison-Wesley (ISBN 978-0321205681) | 確認済み | https://www.oreilly.com/library/view/user-stories-applied/0321205685/ / https://www.pearson.com/en-us/subject-catalog/p/user-stories-applied-for-agile-software-development/P200000009116/9780321205681 | `Cohn "User Stories Applied" 2004 Addison-Wesley ISBN` |
| 7 | ISO/IEC/IEEE 29148:2018 (品質特性 unambiguous/verifiable/singular 等) | 確認済み (規格の実在と品質特性9項目を二次資料で確認。規格本文はペイウォールのため条項番号レベルの照合は未実施 — SKILL.md は条項番号を掲載していないため掲載内容に未検証情報なし) | https://www.iso.org/standard/72089.html / https://standards.ieee.org/standard/29148-2018.html | `"ISO/IEC/IEEE 29148" "unambiguous" "verifiable" "singular" requirement characteristics` |
| 8 | Berry, Kamsties, Krieger (2003). Ambiguity Handbook. University of Waterloo | 確認済み (原文 PDF を直接取得し書誌一致を確認) | https://cs.uwaterloo.ca/~dberry/handbook/ambiguityHandbook.pdf | `Berry Kamsties Krieger "Linguistic Sources of Ambiguity" University of Waterloo` |
| 9 | Wilson, Rosenberg, Hyatt (1997). "Automated Analysis of Requirement Specifications". ICSE '97 (NASA SATC) | 確認済み (会議・著者・weak phrases カタログの存在を確認。ページ番号は未取得 — SKILL.md はページ番号を掲載していないため掲載内容に未検証情報なし) | https://www.semanticscholar.org/paper/Automated-Analysis-of-Requirement-Specifications-Wilson-Rosenberg/2acee78bec617b022807f39d969212a86aa05ef6 | `Wilson Rosenberg Hyatt "Automated Analysis of Requirement Specifications" ICSE 1997 NASA SATC weak phrases` |

結論: 全9件確認済み。反証 (書誌の食い違い) は URL パス2件のみで、いずれも reference 側を修正した。掲載中の DOI・ページ番号 (#1, #2) は一次情報源と一致し、ハルシネーションではないことを確認した。
